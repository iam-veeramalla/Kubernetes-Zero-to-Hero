🌐 Day 2 — How Networking Works Inside Linux
==============================================
Our goal today is to understand what happens when:

              Container A
                  |
                  | network
                  |
              Container B

communicate with each other.

Before Kubernetes networking, we need to understand the Linux pieces underneath it.


First understand the big picture
-------------------------------------
Imagine we create two Linux network namespaces:

      Namespace A                    Namespace B
      -------------                  -------------
         eth0                           eth0
          |                              |
          |                              |
         veth                           veth
          \                              /
           \                            /
            +------ Linux Bridge ------+


  The major components are:

      Network Namespace
             ↓
      veth pair
             ↓
      Linux Bridge
             ↓
      Routing Table
             ↓
      iptables / NAT

Each one has a specific job.


What is a Network Namespace?
----------------------------

A network namespace gives a process its own isolated networking environment.

For example:

      Host
       |
       +-------------------------+
       |                         |
       | Namespace A             |
       |                         |
       | IP: 10.0.0.2            |
       | eth0                    |
       |                         |
       +-------------------------+
       |
       +-------------------------+
       | Namespace B             |
       |                         |
       | IP: 10.0.0.3            |
       | eth0                    |
       |                         |
       +-------------------------+

Without namespaces, all processes would normally use the host's networking.

  With network namespaces:

      Namespace A
          |
          | sees its own
          | interfaces/routes
          |
      Namespace B
          |
          | sees its own
          | interfaces/routes


This is one of the fundamental technologies behind Linux containers.

Let's create two namespaces
---------------------------
Run:

    sudo ip netns add ns1
    sudo ip netns add ns2

Check:

    ip netns list

You should see:

    ns2
    ns1

Now:

    sudo ip netns exec ns1 ip addr

and:

      sudo ip netns exec ns2 ip addr

      
You'll notice they don't have the normal host interfaces.

That's because each namespace has its own networking environment.


But how do we connect the namespace to the host?
------------------------------------------------

This is where the veth pair comes in.

What is a veth pair?
    
    veth = Virtual Ethernet

Think of it like a virtual network cable.

           Virtual Ethernet Cable

       veth1  =================  veth2

Anything entering one end comes out of the other end.

      The two ends are always created together.
      
      veth1 <-----------------> veth2


Create a veth pair
------------------


         sudo ip link add veth-ns1 type veth peer name veth-host1
   
Linux creates two endpoints of one virtual Ethernet cable:

                VETH PAIR
           =====================

      veth-ns1              veth-host1
         │                       │
         │                       │
         ↓                       ↓
      ns1 Network              Host
      Namespace                Network

Now:

      veth-ns1
          |
          | virtual cable
          |
      veth-host1

Put one side inside ns1:

    sudo ip link set veth-ns1 netns ns1

Now the topology is:

      ns1
       |
       | 
       |                veth-host1
       +------------------+
     veth-ns1             |
                          |
                          |
                          |
                         Host

This is extremely similar to what container networking does.

so in step 5 when we are creating Veth it will creating two end points one is veth-ns for network namesapce to connect and another end veth-host for connect to host

      veth-ns1 → endpoint that we put inside the network namespace
      veth-host1 → endpoint that stays on the host
      Together they form one veth pair
      Data sent into one endpoint comes out the other endpoint.

And later, we'll connect veth-host1 to the Linux bridge (br0).



Why do we need a bridge?
-------------------------

Suppose we have:

    ns1
     |
     veth
     |
    Host

Now we create:

    ns2
     |
     veth
     |
    Host

We want:

    ns1  <------->  ns2
    
A Linux bridge can connect them.

Think of a bridge like a virtual switch.


              Linux Bridge
             +-------------+
             |             |
             |    br0      |
             |             |
             +-------------+
                /       \
               /         \
            veth1       veth2
              |           |
             ns1         ns2

This is conceptually similar to:

      Network switch
            |
       +----+----+
       |         |
      PC1       PC2


Create a Linux bridge
---------------------

   
        sudo ip link add br0 type bridge

Bring it up:

    sudo ip link set br0 up

Check:

      ip link show br0

Now we have:

Linux Host

           br0
            |
            |
         Virtual Switch


8. Connect veth to bridge
   ----------------------

Suppose:

    veth-host1
    veth-host2

Connect them:

      sudo ip link set veth-host1 master br0
      sudo ip link set veth-host2 master br0
Now:

                br0
           Linux Bridge
            /        \
           /          \
     veth-host1    veth-host2
          |             |
          |             |
       veth-ns1      veth-ns2
          |             |
         ns1           ns2

This is a very important topology to remember.


Give IP addresses to the namespaces
------------------------------------
Inside ns1:

    sudo ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns1

Inside ns2:

    sudo ip netns exec ns2 ip addr add 10.0.0.3/24 dev veth-ns2

Bring interfaces UP:

    sudo ip netns exec ns1 ip link set veth-ns1 up
    sudo ip netns exec ns2 ip link set veth-ns2 up

Now:

    ns1                         ns2
    
    10.0.0.2                    10.0.0.3
       |                           |
       |                           |
    veth-ns1                    veth-ns2
       |                           |
       +---------- br0 ------------+


Test communication
-----------------
From ns1:

    sudo ip netns exec ns1 ping 10.0.0.3

The packet travels:

    ns1
     |
     | 10.0.0.2
     |
    veth-ns1
     |
     |
    veth-host1
     |
     |
    Linux Bridge
     |
     |
    veth-host2
     |
     |
    veth-ns2
     |
     |
    ns2
     |
    10.0.0.3

So:

    10.0.0.2  ------------->  10.0.0.3

The Linux bridge acts like a virtual switch and forwards the Ethernet frame to the correct veth interface.


Where does the Routing Table come in?
------------------------------------

Now suppose ns1 wants to communicate with:

      10.0.0.3

The kernel checks its routing table.

Run:

    sudo ip netns exec ns1 ip route
    
You should have something conceptually like:

    10.0.0.0/24 dev veth-ns1

This means:

    "10.0.0.0/24 is directly reachable through veth-ns1."

So the kernel doesn't need a router.

It knows:

    Destination: 10.0.0.3
    
    10.0.0.3 belongs to
    10.0.0.0/24
    
    Therefore:
    send through veth-ns1

12. Now imagine different networks

Suppose:

    ns1 = 10.0.0.2

and wants:

    8.8.8.8

Is 8.8.8.8 part of:

10.0.0.0/24

No.

So the routing table might say:

    default via 10.0.0.1

Meaning:

    "I don't know the specific destination, so send the packet to the default gateway."

Flow:

    ns1
     |
    10.0.0.2
     |
    default gateway
     |
    Host
     |
    NAT
     |
    Internet
     |
    8.8.8.8

This is where routing + NAT become important.


What is NAT?
-------------

NAT = Network Address Translation

Containers commonly use private IP addresses:

    10.0.0.2
    10.0.0.3

These aren't normally routable across the public Internet.

Suppose:

    Container
    10.0.0.2
        |
        v
    Host
    192.168.1.100
        |
        v
    Internet

NAT can translate:

      SOURCE
      
      10.0.0.2
         ↓
      192.168.1.100

So the external server sees:

    192.168.1.100

rather than:

    10.0.0.2


14. Where does iptables come in?
    ------------------------------

iptables can implement packet filtering and NAT rules.

Conceptually:

      Container
         |
         v
      Linux networking
         |
         v
      iptables
         |
         +---- ACCEPT
         |
         +---- DROP
         |
         +---- NAT
         |
         v
      Network

For example:

        10.0.0.2
             |
             | packet
             v
        iptables
             |
             | MASQUERADE
             v
        Host IP
             |
             v
        Internet
        
This is why you often see Kubernetes networking commands involving:

iptables

or:

iptables-save


Now connect this to Docker
---------------------------
When you run:

docker run nginx

Docker creates networking components underneath.

Conceptually:

Docker Container
      |
   eth0
      |
    veth
      |
   docker0
      |
    Host
      |
   iptables
      |
   Network

So Docker isn't magically creating networking.

It uses Linux networking technologies:

Network Namespace
        +
veth pair
        +
Linux Bridge
        +
Routing
        +
iptables/NAT


This is where today's lesson becomes extremely important.

A Kubernetes Pod has its own network namespace.

Conceptually:

Pod
 |
 | eth0
 |
veth
 |
Node
 |
CNI
 |
Linux networking

The CNI plugin configures the networking.

Depending on the CNI/plugin and mode, you may see:

veth
bridge
routing
iptables
eBPF

For example, Kubernetes networking can ultimately look conceptually like:

This is where today's lesson becomes extremely important.

A Kubernetes Pod has its own network namespace.

Conceptually:

Pod
 |
 | eth0
 |
veth
 |
Node
 |
CNI
 |
Linux networking

The CNI plugin configures the networking.

Depending on the CNI/plugin and mode, you may see:

veth
bridge
routing
iptables
eBPF

For example, Kubernetes networking can ultimately look conceptually like:

      Pod A
      10.244.1.10
          |
         veth
          |
          +----------------+
                           |
                        Node
                           |
                      Routing /
                      iptables
                           |
                        Network
                           |
                         Pod B
                      10.244.2.10

🧠 Day 2 — Remember This Flow

This is the most important thing to remember:

             Linux Networking

              Application
                   |
                   v
          Network Namespace
                   |
                   v
               eth0
                   |
                   v
                veth
                   |
                   v
            Linux Bridge
             (virtual switch)
                   |
                   v
             Routing Table
                   |
                   v
              iptables
             /         \
          FILTER        NAT
             \         /
              \       /
                Network


  One-line meaning of each:
  | Component         | Simple meaning                    |
| ----------------- | --------------------------------- |
| Network namespace | Gives a container its own network |
| veth pair         | Virtual network cable             |
| Linux bridge      | Virtual switch                    |
| Routing table     | Decides where packets should go   |
| iptables          | Filters/translates packets        |
| NAT               | Changes source/destination IPs    |


Pod
 ↓
Network Namespace
 ↓
veth pair
 ↓
Node
 ↓
CNI
 ↓
Routing / iptables / eBPF
 ↓
Other Pod / Service / Internet

Kubernetes networking becomes much easier to understand.

For Day 2 hands-on, the next step should be to actually build ns1 → veth → bridge ← veth ← ns2 on your Linux machine and use ip addr, ip route, bridge link, and tcpdump to watch the packet move.
