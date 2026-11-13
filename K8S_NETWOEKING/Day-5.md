Day 5 — CNI

Let's keep this very simple and build from yesterday.

Yesterday we learned:

Pod A
10.244.1.10
   ↓
????
   ↓
Pod B
10.244.2.20

Today, CNI is the "????" that helps make this communication possible.

1. What is CNI?

CNI = Container Network Interface

In simple words:

CNI is a standard that allows Kubernetes to connect Pods to a network and give them IP addresses.

When Kubernetes creates a Pod:

Kubernetes
    ↓
Create Pod
    ↓
CNI
    ↓
Give Pod networking
    ↓
Pod gets IP

For example:

Pod A
IP: 10.244.1.10
2. Why does Kubernetes need CNI?

Kubernetes creates Pods, but Kubernetes itself doesn't implement every detail of Linux networking.

For example, Kubernetes needs something to handle:

Give the Pod an IP
Create the Pod's network interface
Connect the Pod to the node network
Configure routes
Enable Pod-to-Pod communication

That's where CNI comes in.

Think:

Kubernetes = "Create this Pod"

CNI = "Okay, I'll connect this Pod to the network."
3. Simple real-world example

Imagine you build an apartment.

Kubernetes = Apartment builder
Pod = Apartment
CNI = Electricity/water connection team

Creating the apartment isn't enough.

You also need to connect it to infrastructure.

Similarly:

Create Pod
    ↓
Need networking
    ↓
CNI
    ↓
Pod connected to network
4. What is a CNI plugin?

CNI plugin is the actual networking implementation.

Some examples:

Calico
Cilium
Flannel
AWS VPC CNI

They all solve the networking problem, but they do it differently.

For example:

Kubernetes
    ↓
CNI
    ↓
Calico

or:

Kubernetes
    ↓
CNI
    ↓
Cilium

or:

Kubernetes
    ↓
CNI
    ↓
Flannel
5. CNI configuration

The CNI plugin needs configuration.

The configuration tells it things like:

Which plugin should I use?
What network should Pods use?
What IP range should I use?
What other networking settings should I apply?

A simplified example:

Pod Network:
10.244.0.0/16

Plugin:
Calico

So conceptually:

CNI configuration
       ↓
"Use this networking plugin"
       ↓
"Use this Pod network"
6. CNI binaries

This sounds complicated, but it's simple.

The CNI binary is the executable program that actually performs the networking operation.

Think:

CNI configuration
        ↓
tells the system WHAT to use

CNI binary
        ↓
actually DOES the networking work

For example:

CNI config
    ↓
Calico configuration
    ↓
Calico CNI binary
    ↓
Configure Pod networking

Don't worry about memorizing filesystem paths yet.

7. Now let's understand the networking flow

This is the most important part of Day 5.

Suppose:

Node 1
   │
   └── Pod A
       10.244.1.10

and:

Node 2
   │
   └── Pod B
       10.244.2.20

Pod A wants to communicate with Pod B.

The simplified flow is:

Pod A
  ↓
CNI / node networking
  ↓
Node 1
  ↓
Network between nodes
  ↓
Node 2
  ↓
CNI / node networking
  ↓
Pod B

Or:

Pod A
10.244.1.10
     ↓
    CNI
     ↓
 Node 1
     ↓
Cluster Network
     ↓
 Node 2
     ↓
    CNI
     ↓
Pod B
10.244.2.20
8. What happens when Pod A is created?

Let's say Kubernetes creates:

Pod A

The process is roughly:

Kubernetes
    ↓
Create Pod
    ↓
Call CNI
    ↓
CNI configures networking
    ↓
Pod gets network interface
    ↓
Pod gets IP

Now:

Pod A
eth0
10.244.1.10

The Pod can now communicate over the cluster network.

9. What happens when Pod A sends traffic?

Suppose:

Pod A = 10.244.1.10
Pod B = 10.244.2.20

Pod A sends:

Source:      10.244.1.10
Destination: 10.244.2.20

The node/networking layer determines where:

10.244.2.20

should go.

Conceptually:

10.244.2.20
      ↓
Pod network
      ↓
Node 2
      ↓
Pod B

The exact mechanism depends on the CNI.

10. Different CNI plugins can work differently

This is important.

Flannel

Often uses an overlay network such as VXLAN.

Conceptually:

Pod A
 ↓
Node 1
 ↓
Overlay/tunnel
 ↓
Node 2
 ↓
Pod B
Calico

Can use routing-based networking and has strong network-policy capabilities.

Conceptually:

Pod A
 ↓
Calico
 ↓
Routing
 ↓
Node 2
 ↓
Pod B
Cilium

Uses eBPF heavily for networking, security, and observability.

Conceptually:

Pod A
 ↓
Cilium/eBPF
 ↓
Network
 ↓
Pod B
AWS VPC CNI

This one is especially interesting in AWS.

Instead of creating a completely separate overlay Pod network in the common case, AWS VPC CNI integrates Pod networking with the AWS VPC networking model.

Conceptually:

Pod
 ↓
AWS VPC CNI
 ↓
AWS VPC networking
 ↓
Other Pod

So the implementation is different, but the goal is the same:

Connect Pods to the network.

11. Don't confuse CNI with Kubernetes

This is a common interview question.

Kubernetes

Responsible for things such as:

Pod creation
Pod scheduling
Pod lifecycle
CNI

Responsible for:

Pod networking
Pod IP
Network interface
Network connectivity

Think:

Kubernetes
    │
    │ "Create Pod"
    ↓
   Pod
    │
    │ "Connect me to network"
    ↓
   CNI
    │
    ↓
Network
12. The easiest way to remember CNI

Think of CNI as the networking team for Pods.

Kubernetes → creates Pod

CNI → connects Pod

CNI plugin → actual networking implementation

And the most important flow:

        Pod A
          ↓
         CNI
          ↓
        Node 1
          ↓
    Cluster Network
          ↓
        Node 2
          ↓
         CNI
          ↓
        Pod B
🔥 Interview answer

If someone asks:

"What is CNI in Kubernetes?"

Say:

CNI stands for Container Network Interface. Kubernetes uses CNI plugins to provide networking for Pods, including assigning Pod IP addresses, creating network interfaces, and enabling Pod-to-Pod communication across nodes. Examples include Calico, Cilium, Flannel, and AWS VPC CNI.

For Day 5, don't go deep into veth, bridge, VXLAN, eBPF, or routing tables yet. First remember the big picture:

Kubernetes creates the Pod → CNI connects the Pod → CNI/networking makes Pod-to-Pod communication possible.
