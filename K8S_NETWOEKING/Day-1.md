Kubernetes Networking — Day 1: Networking Fundamentals
========================================================

Today we will not start with Kubernetes yet.

First, we need to understand the networking concepts that Kubernetes is built on. If these are clear, CNI, Services, kube-proxy, CoreDNS, and NetworkPolicy will become much easier.

What is a network?
----------------------
A network is a way for devices to communicate with each other.

For example:

         Laptop
            |
            |
         Router
            |
            +------ Server 1
            |
            +------ Server 2

The laptop can communicate with Server 1 and Server 2 through the network.

In a Kubernetes environment:

      Node 1
         |
         +---- Pod A
         +---- Pod B
      
      
      Node 2
         |
         +---- Pod C
         +---- Pod D

The Pods also need a network so they can communicate.

What is an IP address?
-----------------------
An IP address identifies a device/interface on a network.

An IP address is a logical address assigned to a network interface, used to identify and communicate with a device/interface on an IP network.

                  Server
                    │
                    └── Network Interface (eth0)
                            │
                            └── IP: 192.168.1.10


Example:

      Server A → 192.168.1.10
      Server B → 192.168.1.20

If Server A wants to communicate with Server B:

      192.168.1.10
            |
            | network
            ↓
      192.168.1.20

Think of an IP address like a house address.

If I want to send something to your house, I need to know your address.

Similarly:

         Network packet
               ↓
         Destination IP
               ↓
         192.168.1.20


IPv4
--------
Most Kubernetes networking discussions use IPv4.

Example:

      192.168.1.10

IPv4 contains 4 numbers separated by dots.

      192 . 168 . 1 . 10

Each section can range from:

      0 - 255

So this is valid:

      192.168.1.10

This is not valid:

         192.168.1.300

         
Public IP vs Private IP
-------------------------
This is very important in AWS and Kubernetes.

Private IP

Used inside private networks.

Examples:

         10.0.0.10
         172.16.10.20
         192.168.1.50
Public IP

Used to communicate over the public Internet.

Example:

      13.x.x.x
      3.x.x.x

Imagine an AWS VPC:

Internet
   |
   |
Public IP
   |
EC2
   |
Private IP
10.0.1.10

The EC2 instance can have:

      Public IP  → Internet communication
      Private IP → VPC communication


What is a subnet?
-------------------
Suppose we have:

      10.0.0.0/24

This represents a network containing IP addresses from approximately:

      10.0.0.0
            ↓
      10.0.0.255

We can divide a large network into smaller networks.

For example:

      10.0.0.0/16

can be divided into:

      10.0.1.0/24
      10.0.2.0/24
      10.0.3.0/24

In AWS:

      VPC
      10.0.0.0/16
            |
            +---- Public Subnet
            |     10.0.1.0/24
            |
            +---- Private Subnet
                  10.0.2.0/24

This becomes very important when we study Kubernetes nodes and Pod CIDRs.

What is a port?
----------------
An IP address identifies the machine/interface.

A port identifies the application/service on that machine.

For example:

Server
10.0.1.10

Maybe this server is running:

      SSH       → 22
      HTTP      → 80
      HTTPS     → 443
      MySQL     → 3306

So:

      10.0.1.10:22

means:

Connect to port 22 on the machine with IP 10.0.1.10.

And:

10.0.1.10:80

means:

Connect to the HTTP application on that machine.

IP + Port
-------------
This is a very important combination.

Think:

      IP   = Which machine?
      Port = Which application?

For example:

      10.0.1.10:8080

means:

      Machine → 10.0.1.10
      Application → port 8080

Later in Kubernetes:

Service IP : Service Port

and:

Pod IP : Container Port

will make much more sense.

TCP vs UDP
------------
Two important transport protocols are:

      TCP
      UDP
TCP

TCP provides reliable communication.

Example:

Client
  |
  | TCP
  ↓
Server

TCP makes sure data is delivered reliably and in order.

Common examples:

      HTTP
      HTTPS
      SSH
      MySQL
UDP

UDP is simpler and faster but does not provide TCP-style delivery guarantees.
UDP itself doesn't say: "Packet 2 was lost, let me retransmit it."

The application/protocol running on top of UDP can implement its own reliability if needed.

Common examples include:

DNS
DHCP
streaming/real-time traffic

For Kubernetes, remember:

DNS commonly uses UDP port 53.

TCP vs UDP
-----------
| TCP                   | UDP                          |
| --------------------- | ---------------------------- |
| Connection-oriented   | Connectionless               |
| Reliable              | No delivery guarantee        |
| Ordered delivery      | No ordering guarantee        |
| Retransmits lost data | Doesn't retransmit by itself |
| 3-way handshake       | No TCP-style handshake       |
| More overhead         | Lower overhead               |
| Generally slower      | Generally faster             |
| File transfer         | Real-time traffic            |

What is a MAC address?
-------------------------
An IP address is a logical network address.

A MAC address identifies a network interface at the data-link layer.

A MAC address is a Layer-2 hardware/link-layer address associated with a network interface.


                  Server
                    │
                    └── eth0
                         │
                         ├── MAC: 00:1A:2B:3C:4D:5E
                         │
                         └── IP: 192.168.1.10



MAC address → identity/address of the network interface at Layer 2
IP address  → logical network address used for communication/routing



Example:

MAC:
02:42:ac:11:00:02

You normally don't need to memorize MAC addresses.

Just understand:

IP address
   ↓
Logical network address


MAC address
   ↓
Network interface address

Later, when we study ARP, bridges, veth pairs, and containers, MAC addresses become important.

What is a network interface?
-----------------------------
A server needs something to connect to a network.

That is a network interface.

On Linux, you might see:

                  ip addr

You may see:

                  eth0
                  ens5
                  lo

For example:

                  ens5
                    |
                    +---- 10.0.1.10

The interface is basically the connection between the machine and the network.

What is localhost?
----------------------
localhost means:

This same machine.

Usually:

                  127.0.0.1

Example:

                  Application
                      |
                      ↓
                  127.0.0.1:8080

This means the application is listening only on the local machine.

It doesn't automatically mean other machines can access it.

12. What is routing?

This is one of the most important concepts for Kubernetes networking.

Imagine:

                  Server A
                  10.0.1.10
                      |
                      |
                      ↓
                  Router
                      |
                      |
                      ↓
                  Server B
                  10.0.2.10

Server A needs to know:

Where should I send traffic destined for 10.0.2.10?

The answer comes from the routing table.

On Linux:

ip route

You might see something like:

10.0.1.0/24 dev eth0
default via 10.0.1.1 dev eth0

Meaning roughly:

                  10.0.1.0/24
                      ↓
                  Directly reachable through eth0

and:

default via 10.0.1.1
    ↓
For destinations I don't know,
send traffic to 10.0.1.1

What is a default gateway?
--------------------------
Suppose your server has:

IP:
10.0.1.10

and wants to reach:

8.8.8.8

8.8.8.8 is not on its local network.

So the server sends traffic to its:

Default Gateway

Example:

Server
10.0.1.10
   |
   ↓
Gateway
10.0.1.1
   |
   ↓
Internet
   |
   ↓
8.8.8.8

Think of the gateway as:

The door through which traffic leaves your network.

What is DNS?
-------------
DNS converts names into IP addresses.

Instead of remembering:

142.250.x.x

you use:

google.com

Flow:

Application
    |
    | "What is google.com?"
    ↓
DNS Server
    |
    | "google.com = IP address"
    ↓
IP address

In Kubernetes, DNS becomes extremely important.

For example:

my-service

can resolve to a Kubernetes Service IP.

We'll study CoreDNS later.

What is NAT?
------------
NAT means:

Network Address Translation

Suppose a private server has:

10.0.1.10

It wants to access the Internet.

Private IPs aren't directly routable over the public Internet.

So NAT can translate:

10.0.1.10
     ↓
NAT
     ↓
Public IP
     ↓
Internet

In AWS private subnets, this is commonly done through a:

NAT Gateway

Later, Kubernetes networking will also involve NAT in some traffic paths.

Now connect these concepts to Kubernetes
------------------------------------------
Suppose we have:

Kubernetes Node 1
10.0.1.10


Kubernetes Node 2
10.0.2.10

Each node has its own network interface.

Then Kubernetes creates Pods:

Node 1
10.0.1.10
   |
   +---- Pod A
         10.244.1.10


Node 2
10.0.2.10
   |
   +---- Pod B
         10.244.2.10

Now we have a problem:

Pod A
10.244.1.10
     |
     | ???
     ↓
Pod B
10.244.2.10

How does this packet travel?

Who creates the Pod network?

Who assigns the Pod IP?

How does Node 1 know where 10.244.2.10 is?

How does the packet move from Node 1 to Node 2?

That's where these concepts come together:

Pod
 ↓
Network Namespace
 ↓
veth pair
 ↓
Bridge / interface
 ↓
Routing
 ↓
CNI
 ↓
Node network
 ↓
Other Node
 ↓
Pod

We will not jump into this yet.

First make sure today's fundamentals are solid.

Day 1 — What you should remember
Concept	Simple meaning
Network	Allows devices to communicate
IP	Identifies a network endpoint
Subnet	A smaller network
Port	Identifies an application
TCP	Reliable transport
UDP	Lightweight transport
MAC	Network interface address
Interface	Connection to a network
Routing	Decides where packets go
Gateway	Door to another network
DNS	Name → IP
NAT	Changes source/destination addressing
The most important mental model
IP
 ↓
Which machine?


Port
 ↓
Which application?


Route
 ↓
Where should the packet go?


Gateway
 ↓
How do I reach another network?


DNS
 ↓
What IP belongs to this name?


NAT
 ↓
How can private addressing communicate with another network/Internet?
Mini test for you

Don't look back at the explanation. Answer these in your own words:

What is an IP address?
What is a port?
What is the difference between IP and port?
What is a subnet?
What is a default gateway?
What does routing do?
What does DNS do?
What is NAT?
What is the difference between TCP and UDP?
If Pod A has IP 10.244.1.10 and Pod B has IP 10.244.2.20, what basic networking problem must Kubernetes solve?

Send me your answers. I'll correct them like an interview and then we'll move to Day 2: Linux networking — network namespaces, veth pairs, bridges, routing tables and iptables.
