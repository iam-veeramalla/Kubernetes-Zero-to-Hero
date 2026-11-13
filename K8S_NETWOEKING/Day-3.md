🐳 Day 3 — Container Networking
================================

We will understand this architecture:

                          HOST MACHINE
          ┌──────────────────────────────────────────────┐
          │                                              │
          │              docker0 bridge                  │
          │            172.17.0.1/16                    │
          │              │             │                 │
          │              │             │                 │
          │          veth-host     veth-host             │
          │              │             │                 │
          │              │             │                 │
          │        eth0/container  eth0/container        │
          │              │             │                 │
          │        ┌─────┴─────┐ ┌─────┴─────┐           │
          │        │ Container │ │ Container │           │
          │        │ 172.17.0.2│ │ 172.17.0.3│           │
          │        └───────────┘ └───────────┘           │
          │                                              │
          └──────────────────────────────────────────────┘
                                 │
                                 │ NAT
                                 ▼
                              Internet

Don't worry about understanding the whole diagram immediately. We'll build it piece by piece.



What happens when Docker creates a container?
----------------------------------------------

Suppose we run:

    docker run -d --name app nginx

Docker needs to give this container networking.

Conceptually Docker does something like:

      Create Container
            ↓
      Create Network Namespace
            ↓
      Create veth pair
            ↓
      Put one end inside container
            ↓
      Put other end on host
            ↓
      Connect host end to docker0 bridge
            ↓
      Give container an IP
            ↓
      Configure routing
            ↓
      Configure NAT
This is very similar to what you learned in Day 2.


Container gets its own Network Namespace
------------------------------------------
Remember Linux Network Namespace?

It gives a process its own:

      Network interfaces
      IP addresses
      Routing table
      iptables rules

Docker uses this concept.

So:

      Host
      │
      ├── eth0
      ├── docker0
      └── veth...
          
      Container
      │
      └── eth0

The container doesn't directly use the host's eth0.

It gets its own network namespace.


Container gets its own IP
-----------------------------
For example:

    Container
        │
        └── eth0
              │
              └── 172.17.0.2

Another container:

    Container
        │
        └── eth0
              │
              └── 172.17.0.3

So:

      Container A → 172.17.0.2
      Container B → 172.17.0.3


These are container IP addresses.



How does the container connect to the host?
------------------------------------------------
This is where the veth pair comes in.

Remember from Day 2:

      A veth pair is like a virtual network cable with two ends.

Docker creates:

    Container
       │
       │ eth0
       │
       ▼
    veth pair
       │
       │
       ▼
    Host

More specifically:

      Container namespace              Host namespace
      
          eth0
            │
            │
            ▼
         veth-container ═══════════ veth-host
                                        │
                                        ▼
                                     docker0

One end lives inside the container.

The other end lives on the host.


What is docker0?
------------------

Now we need something to connect multiple containers.

Docker creates a Linux bridge called:

      docker0

Think of it like a virtual switch.

                 docker0
              Linux Bridge
             ┌───────────┐
             │           │
             └─┬─────┬───┘
               │     │
             veth   veth
               │     │
               ▼     ▼
           Container Container
           172.17.0.2 172.17.0.3

This allows containers connected to the same Docker bridge network to communicate.



Container → Container communication

Suppose:

    Container A
    172.17.0.2

wants to communicate with:

    Container B
    172.17.0.3

The flow is approximately:

      Container A
      172.17.0.2
           │
           ▼
          eth0
           │
           ▼
      veth pair
           │
           ▼
         docker0
           │
           ▼
      veth pair
           │
           ▼
          eth0
           │
           ▼
      Container B
      172.17.0.3

Notice:

The internet is not involved.

What is the container's gateway?
---------------------------------

The Docker bridge itself normally has an IP such as:

    docker0
    172.17.0.1

The container might have:

    Container IP:
    172.17.0.2
    
    Gateway:
    172.17.0.1

So:

Container
172.17.0.2
     │
     │ default route
     ▼
172.17.0.1
     │
     ▼
docker0

You can think of docker0 as the container network's gateway.


Container → Internet
--------------------


Now the interesting part.

Suppose the container runs:

    curl https://google.com

The destination is outside the Docker network.

The flow becomes:

      Container
      172.17.0.2
           │
           ▼
      eth0
           │
           ▼
      veth
           │
           ▼
      docker0
      172.17.0.1
           │
           ▼
      Host routing
           │
           ▼
      NAT
           │
           ▼
      Host eth0
           │
           ▼
      Internet

This is where NAT comes in.


Why do we need NAT?
---------------------

The container has:

    172.17.0.2

This is a private Docker-network IP.

The internet cannot directly route back to that private container IP.

So the host performs NAT.

Conceptually:

Before NAT:

    172.17.0.2
         │
         ▼
    Internet

Host changes the source address:

    172.17.0.2
         ↓
    Host's IP

So the internet sees something like:

    Host Public/Private IP
            ↓
         Internet
When the response comes back, the host's NAT state allows the response to be sent back to the correct container.


Now understand Port Mapping
--------------------------------------
Suppose your container runs Nginx:

Container
172.17.0.2
Port 80

You want users to access it from the host.

You run:

docker run -d -p 8080:80 nginx

This means:

    Host port       Container port
        8080   →        80

So:

      Browser
         │
         │ http://HOST_IP:8080
         ▼
      Host
      Port 8080
         │
         │ Docker port forwarding/NAT
         ▼
      Container
      172.17.0.2:80
         │
         ▼
      Nginx


Very important: -p 8080:80
---------------------------
Remember this:

    -p HOST_PORT:CONTAINER_PORT

Therefore:

    docker run -p 8080:80 nginx

means:

    Host:       8080
         ↓
    Container: 80

It does not mean the application inside the container runs on port 8080.

Nginx is still listening on:

    80

Container port vs host port
-------------------------------
This distinction is extremely important.


      HOST
      192.168.1.10
           │
           │ :8080
           ▼
       Docker
           │
           │ forwarding
           ▼
      CONTAINER
      172.17.0.2
           │
           │ :80
           ▼
       Nginx

So:

    8080 → Host port
    80   → Container/application port

Let's connect everything

Now we can build the complete Docker networking picture:


                         INTERNET
                            │
                            │
                         Host IP
                            │
                     ┌──────┴──────┐
                     │     NAT     │
                     └──────┬──────┘
                            │
                         docker0
                      172.17.0.1
                     Linux Bridge
                     ┌──────┴──────┐
                     │             │
                   veth          veth
                     │             │
                     ▼             ▼
               Container A    Container B
               172.17.0.2     172.17.0.3
                     │             │
                    eth0          eth0



What we will do hands-on

Let's not jump straight into theory.

We'll verify every piece on your Linux machine.

Step 1 — Check Docker
docker version
Step 2 — Check Docker networks
docker network ls

You should normally see something like:

NETWORK ID     NAME      DRIVER
xxxxx          bridge    bridge
xxxxx          host      host
xxxxx          none      null
Step 3 — Inspect the bridge network
docker network inspect bridge

Look for:

Subnet
Gateway
Containers

For example:

Subnet: 172.17.0.0/16
Gateway: 172.17.0.1
Step 4 — Run a container
docker run -d --name app nginx
Step 5 — Check its IP
docker inspect app

You'll find something like:

172.17.0.2
Step 6 — Check the host bridge
ip addr show docker0
Step 7 — Check veth interfaces
ip link

You'll see interfaces related to the container.
