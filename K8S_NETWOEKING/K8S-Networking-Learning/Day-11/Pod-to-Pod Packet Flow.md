Pod-to-Pod Packet Flow:
=======================

Today's goal: Understand how a network packet travels from Pod A to Pod B in Kubernetes, especially when the Pods are on different worker nodes in an AWS EKS cluster.

By the end, you'll be able to explain the packet path and identify where to investigate when communication fails.

<img width="987" height="607" alt="image" src="https://github.com/user-attachments/assets/2a55a635-3358-4add-9550-e7df8ac89713" />

<img width="985" height="782" alt="image" src="https://github.com/user-attachments/assets/aa1bcbd9-0b6a-4324-bd0a-916c7fd99d0e" />

What happens at each step?

Step 1 — Pod A sends the packet
-------------------------------
The application inside Pod A creates an HTTP request for 10.0.2.20:8080.

The packet contains information such as:

    - Source IP: 10.0.1.10
    - Destination IP: 10.0.2.20
    - Destination port: 8080
    - Protocol: TCP

Step 2 — Linux decides where to send it
----------------------------------------

The packet enters the networking path associated with Pod A.

Linux networking and routing determine the next hop and outgoing interface.

Step 3 — Packet travels through the VPC
----------------------------------------

Because Pod B is on another worker node, the packet travels through the AWS VPC network.

AWS networking controls, such as security groups and network ACLs, can affect whether the traffic is allowed.

Step 4 — Worker Node 2 receives it
-------------------------------------
Worker Node 2 receives the packet and its Pod networking delivers it to Pod B.

Step 5 — Pod B's application receives it
----------------------------------------------
If the application is listening on 10.0.2.20:8080 (or an appropriate wildcard address),
and the network path allows the connection, it can process the request and send a response.


Practical: inspect the packet path:
-----------------------------------

Let's use three useful commands.

Command 1: Find the Pods and their nodes

    kubectl get pods -o wide

This displays Pod IPs and the worker nodes where the Pods are running.

Command 2: Check the route to Pod B

On Worker Node 1:

    ip route get 10.0.2.20


This helps identify the route Linux selects for Pod B's IP.

Remember: a route being present does not guarantee that the connection will succeed.

Command 3: Capture traffic using tcpdump

On Worker Node 1:

    sudo tcpdump -ni any host 10.0.2.20 and port 8080


This captures matching traffic visible on the node's interfaces.

Then, from Pod A, generate traffic:
    
    kubectl exec -it pod-a -- \
      curl -v --connect-timeout 5 http://10.0.2.20:8080


Use your actual Pod name and IP. The tcpdump command requires the utility to be installed and sufficient permissions.
Capturing on any can show the same packet more than once.

<img width="958" height="422" alt="image" src="https://github.com/user-attachments/assets/b9ae02fc-cc70-460d-be19-1c55ab66000d" />

