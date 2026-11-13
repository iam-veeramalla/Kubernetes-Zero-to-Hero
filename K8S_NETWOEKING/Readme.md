Kubernetes Networking — From Scratch
Our learning path

We'll follow this order:

Day 1 — Networking fundamentals
What is an IP address?
Private vs public IP
MAC address
Port
TCP vs UDP
DNS
Routing
NAT
Network interface
Linux network namespaces
Bridge
Day 2 — How networking works inside Linux

This is extremely important for Kubernetes.

We'll learn:

Network namespace
veth pair
Linux bridge
Routing table
iptables
NAT
How two namespaces communicate

You'll actually understand what happens underneath a container.

Day 3 — Container networking

We'll move from Linux networking to containers:

Container
   ↓
Network Namespace
   ↓
veth pair
   ↓
Linux Bridge
   ↓
Host Network

We'll understand:

Docker networking
Container IP
Bridge network
Port mapping
docker0
NAT
Day 4 — Kubernetes Pod networking

Now Kubernetes starts.

We'll understand the most important Kubernetes networking rule:

Every Pod gets its own IP address, and Pods should be able to communicate with other Pods without NAT.

We'll understand:

Pod A
10.244.1.10
      ↓
      ?
      ↓
Pod B
10.244.2.20

And answer:

How can Pod A communicate with Pod B when they are on different nodes?

Day 5 — CNI

Then we'll understand:

What is CNI?
Why Kubernetes needs CNI
CNI plugin
CNI configuration
CNI binaries
CNI networking flow
Calico
Cilium
Flannel
AWS VPC CNI

Especially:

Pod
 ↓
CNI
 ↓
Node Network
 ↓
Other Node
 ↓
Pod
Day 6 — Kubernetes Services

Then:

ClusterIP
NodePort
LoadBalancer
Service IP
kube-proxy
iptables mode
IPVS
Endpoint / EndpointSlice

We'll understand:

Client
  ↓
Service
10.96.10.20
  ↓
Pod 1
Pod 2
Pod 3

And most importantly:

How does the Service IP actually reach a Pod?

Day 7 — DNS

We'll cover:

CoreDNS
kube-dns
Service DNS
Pod DNS
/etc/resolv.conf
DNS search domains
ndots
CoreDNS configuration

For example:

my-service
    ↓
CoreDNS
    ↓
10.96.100.20
Day 8 — Ingress

Then:

Internet
   ↓
Load Balancer
   ↓
Ingress Controller
   ↓
Service
   ↓
Pod

We'll understand:

Ingress
Ingress Controller
NGINX Ingress
ALB Ingress
Host-based routing
Path-based routing
TLS termination
Day 9 — NetworkPolicy

We'll learn Kubernetes security networking:

Pod A ───────→ Pod B
       ALLOW


Pod C ───────→ Pod B
       DENY

Topics:

NetworkPolicy
Ingress rules
Egress rules
Namespace selectors
Pod selectors
IP blocks
Default deny
Day 10 — Advanced Kubernetes networking

Finally:

kube-proxy
iptables
IPVS
eBPF
Cilium
Service routing
Node routing
Overlay vs underlay
VXLAN
AWS VPC CNI
Pod-to-Pod traffic
Pod-to-Service traffic
Pod-to-Internet traffic
External-to-Pod traffic
First, understand the big picture

Before learning any commands, remember this architecture:

                    Kubernetes Cluster


        ┌─────────────────────────────────────┐
        │                                     │
        │   Node 1                Node 2      │
        │                                     │
        │  ┌─────────┐          ┌─────────┐  │
        │  │ Pod A   │          │ Pod C   │  │
        │  │10.0.1.10│          │10.0.2.10│  │
        │  └────┬────┘          └────┬────┘  │
        │       │                    │       │
        │  ┌────┴────┐          ┌────┴────┐  │
        │  │ Network │          │ Network │  │
        │  │         │          │         │  │
        │  └─────────┘          └─────────┘  │
        │                                     │
        └─────────────────────────────────────┘

There are four major networking problems Kubernetes has to solve:

1. Pod → Pod
Pod A ─────────→ Pod B
2. Pod → Service
Pod
 ↓
Service
 ↓
Pod
3. External user → Service
Internet
   ↓
LoadBalancer / NodePort
   ↓
Service
   ↓
Pod
4. Pod → Internet
Pod
 ↓
Node
 ↓
NAT
 ↓
Internet

If you understand these four flows, Kubernetes networking becomes much easier.

Start with the most basic concept: IP address

Suppose you have two servers:

Server A → 192.168.1.10
Server B → 192.168.1.20

When Server A wants to communicate with Server B:

192.168.1.10
      ↓
   Network
      ↓
192.168.1.20

The IP address identifies where the network device is reachable.

Now imagine a Kubernetes node:

Node IP
192.168.1.10

Inside that node we have:

Pod A → 10.244.1.10
Pod B → 10.244.1.11

So now we have different IP networks:

Node network:


192.168.1.0/24




Pod network:


10.244.0.0/16

The question becomes:

How does traffic from 10.244.1.10 reach 10.244.2.20 when those Pods are on different nodes?

That is the fundamental problem Kubernetes networking solves.

And that's exactly where we'll go next.

Your first mental model

Keep this in your head:

Kubernetes Networking
        │
        ├── Pod Networking
        │
        ├── Service Networking
        │
        ├── DNS
        │
        ├── Ingress
        │
        └── Network Security

But underneath all of them:

IP
 ↓
Route
 ↓
Interface
 ↓
Bridge
 ↓
iptables / eBPF
 ↓
CNI

We'll learn from the bottom upward, so you don't have to memorize Kubernetes networking commands without understanding what they actually do.
