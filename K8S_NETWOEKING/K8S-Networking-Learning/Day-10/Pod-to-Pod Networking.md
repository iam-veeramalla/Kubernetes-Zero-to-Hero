Day 10 — Pod-to-Pod Networking in Kubernetes:
=============================================

Today we'll understand how one Pod communicates with another Pod in Kubernetes.

We'll cover two situations:

    1. Pod-to-Pod communication on the same worker node.
    2. Pod-to-Pod communication on different worker nodes.

Before starting, remember what we learned in Days 5–9:

    - AWS VPC CNI assigns IP addresses to Pods.
    - Pods normally receive IP addresses from the VPC subnet.
    - ENIs provide networking capacity to EC2 nodes.
    - Prefix delegation can provide more Pod IP capacity.
Now let's understand what happens after a Pod receives its IP address.

What is Pod-to-Pod communication?
---------------------------------

Suppose you have two Pods:

    - Pod A: 10.0.1.10
    - Pod B: 10.0.1.20
Pod A wants to send a request to Pod B.

For example, Pod A runs:

    curl http://10.0.1.20:8080
The request must travel from Pod A to Pod B over the network.

Kubernetes networking allows Pods to communicate using their IP addresses, subject to network policies, routing, security rules, and application configuration.

Case 1 — Both Pods are on the same worker node
----------------------------------------------

<img width="698" height="407" alt="image" src="https://github.com/user-attachments/assets/846acb56-4113-4867-9c8f-e1cd7bf0e839" />

What happens?

    1. Pod A sends a network request to Pod B's IP address.
    2. The Linux networking stack on the worker node determines how to reach the destination.
    3. The AWS VPC CNI-configured networking path delivers the packet to Pod B.
    4. Pod B receives the request, provided the application is listening and network controls allow it.
Important: In AWS VPC CNI, don't assume that all Pod-to-Pod traffic passes through a Linux bridge. 

The exact networking path depends on the CNI implementation and configuration.

* Same-node communication in simple words
* Both Pods are connected to the same worker node's networking environment.
* The worker node handles packet delivery between their Pod network interfaces. The packet does not need to travel out to another EC2 instance and back.

Case 2 — Pods are on different worker nodes
--------------------------------------------

<img width="972" height="477" alt="image" src="https://github.com/user-attachments/assets/745218bf-c910-4ca6-8701-e38e3a0eba25" />

What happens?

    1. Pod A sends a request to Pod B's IP address.
    2. Worker Node 1 forwards the packet toward the destination using its networking configuration.
    3. The packet travels through the AWS VPC network.
    4. The VPC network routes it to Worker Node 2's ENI.
    5. Worker Node 2 delivers the packet to Pod B.


Because standard AWS VPC CNI assigns VPC-routable addresses to Pods, Pods on different nodes can communicate directly using their Pod IP addresses, subject to routing and security controls.

Important: This is different from overlay networking, where Pod traffic may be encapsulated inside another network packet.

<img width="952" height="432" alt="image" src="https://github.com/user-attachments/assets/6da14cf2-93ca-494d-898b-0ba4518abcdf" />

<img width="967" height="782" alt="image" src="https://github.com/user-attachments/assets/27583886-7d2e-4b92-808c-221b5a0251c3" />

<img width="1012" height="407" alt="image" src="https://github.com/user-attachments/assets/317be94c-1b4a-4f9f-87e4-6253e3a08055" />

<img width="1000" height="518" alt="image" src="https://github.com/user-attachments/assets/a3a7ab05-75f6-4fc1-adb1-9ce8afb1ba20" />

<img width="832" height="610" alt="image" src="https://github.com/user-attachments/assets/1e8f0082-412c-47f1-b158-9dea110255db" />

