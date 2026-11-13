1. Every Pod gets an IP
=======================
Imagine two Pods:

Pod A
IP: 10.244.1.10

Pod B
IP: 10.244.2.20

Just like two computers can have different IP addresses, each Kubernetes Pod gets its own IP.

2. Same node

Suppose both Pods are on Node 1:

Node 1
│
├── Pod A → 10.244.1.10
│
└── Pod B → 10.244.1.20

Pod A wants to talk to Pod B:

10.244.1.10
      ↓
10.244.1.20

Simple. They are on the same node.

3. Different nodes — the important part

Now:

Node 1                    Node 2
│                         │
├── Pod A                 ├── Pod B
│   10.244.1.10           │   10.244.2.20

Pod A wants to talk to Pod B:

Pod A
10.244.1.10
     │
     │ "I want to reach 10.244.2.20"
     ↓
   Node 1
     │
     │ Kubernetes network
     ↓
   Node 2
     │
     ↓
Pod B
10.244.2.20

This is what Kubernetes networking/CNI handles.

4. Think of it like a courier 📦

Imagine:

Pod A = House A
Pod B = House B

Pod A wants to send a package to:

House B
10.244.2.20

The delivery system looks at the address and figures out:

10.244.2.20
      ↓
This Pod is on Node 2
      ↓
Send package to Node 2
      ↓
Deliver to Pod B

The Pod doesn't need to worry about how the package travels.

5. What is CNI?

Very simply:

CNI is responsible for connecting Pods to the Kubernetes network.

For example:

Kubernetes creates Pod
        ↓
CNI gives Pod networking
        ↓
Pod gets IP
        ↓
Pod can communicate

Examples:

Calico
Cilium
Flannel
AWS VPC CNI
6. What does "without NAT" mean?

This is important.

Pod A:

10.244.1.10

sends to Pod B:

10.244.2.20

The destination remains:

10.244.2.20

We don't want Kubernetes to unnecessarily change it to something like:

Node 1 IP → Node 2 IP → Pod B

Instead, Kubernetes networking provides connectivity to the actual Pod IP.

7. Just remember this diagram
          Kubernetes Cluster

   Node 1                 Node 2
┌─────────────┐       ┌─────────────┐
│             │       │             │
│   Pod A     │       │   Pod B     │
│10.244.1.10  │ ────> │10.244.2.20  │
│             │       │             │
└─────────────┘       └─────────────┘
        ↑                     ↑
        └──── CNI/network ────┘
One-line interview answer

Every Kubernetes Pod gets its own IP, and the CNI networking plugin provides connectivity so Pods can communicate directly across nodes without NAT.

Main thing to understand today:
Pod A → Pod IP → CNI/network → other Node → Pod B

That's the core of Kubernetes Pod networking.
