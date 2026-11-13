Day 1 — Understand how a Pod communicates
=========================================

1. First, imagine two Pods
   
        Suppose your EKS cluster has two Pods running on different EC2 worker nodes.
        Imagine Pod A is a Node.js application that needs to call an API running in Pod B.
        Pod A sends a request to Pod B's IP address. How does that request reach the other Pod?

* Let's understand it step by step.


2. What happens inside Pod A?

* You already know about network namespaces and veth pairs. Let's connect them to this example.

      1. Application
      Sends a request to Pod B
      
      2. Pod network namespace
      Has its own network interface and routes
      
      3. Host-side networking
      Connects the Pod to the node's network path
      
      4. AWS VPC network
      Carries traffic toward the destination node

* Here's what each part means:

      - Application: Creates the request.
      - Network namespace: Gives the Pod its own network view, including interfaces and routes.
      - Host-side networking: Connects the Pod's network to the worker node's networking.
      - AWS VPC: Provides network connectivity between the worker nodes.


Important: AWS VPC CNI doesn't require every Pod to connect through a Linux bridge. The exact networking path depends on the CNI configuration.


3. What is CNI doing?
   
* Think of CNI as the networking setup mechanism for Pods.
* When Kubernetes creates a Pod, the container runtime invokes the configured CNI plugin to set up the Pod's networking.
* In the default AWS VPC CNI setup, the main components have different responsibilities:

      CNI plugin
      Configures the Pod's network interface and connectivity.
      
      IPAMD
      Manages ENIs and available IP addresses or prefixes for Pods.
      
      Linux kernel and AWS network
      Forward packets along the configured path.

The key point is that assigning an IP address and delivering packets are two different jobs.
For example, giving Pod A the address 10.0.1.25 doesn't automatically guarantee it can communicate with Pod B. The networking path and applicable security controls must also allow the traffic.

4. What is routing?

* Routing is simply deciding where a packet should go next.
* Imagine you want to travel from your home to another city. You check the route and choose the road that takes you there.
* Linux does something similar.
* When Pod A wants to send data to Pod B, the networking stack checks its routing table to determine how to reach the destination.

* A simplified example:

        Destination: 10.0.2.35
        
        Linux checks its routing table
                  |
                  v
        Which route matches this IP?
                  |
                  v
        Choose the next hop/interface
                  |
                  v
        Send the packet onward

* The route tells Linux where to send the packet. It doesn't guarantee the destination will receive it.
* For example, traffic can still be blocked by a security group or NetworkPolicy, or fail because the application isn't listening on the required port.
