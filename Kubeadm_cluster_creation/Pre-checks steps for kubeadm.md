Setup a Multi Node Kubernetes Cluster Using Kubeadm:
=====================================================

* In this kubeadm setup we are using Compute as AWS EC2 instances
  1. Create a SG in aws with the required pots for the k8s components communication for that refer the below link:
     
           https://kubernetes.io/docs/reference/networking/ports-and-protocols/

  2. created 3 ec2 instances in the aws cloud.
  3. 1 Master and 2 worker Nodes.
  4. If you are using AWS EC2, you need to disable source destination check for the VMs using the:
     
            https://docs.aws.amazon.com/vpc/latest/userguide/work-with-nat-instances.html#EIP_Disable_SrcDestCheck

     * why we are doing this?
       
                  What does Source/Destination Check do?
                  
                  AWS verifies that:
                  
                  The instance is the source of the packet it is sending, or
                  The instance is the destination of the packet it is receiving.
                  
                  If neither is true, AWS drops the packet before it can be forwarded.
                  
                  Why is this a problem for a NAT instance?
                  
                  A NAT (Network Address Translation) instance acts like a router for instances in a private subnet.
                  
                  Example:
                  Private EC2 (10.0.2.10)
                            |
                            v
                  NAT Instance (10.0.1.10)
                            |
                            v
                  Internet (8.8.8.8)
                  
                  Suppose the private EC2 wants to access Google DNS (8.8.8.8).

                  The packet arriving at the NAT instance looks like:
                  Source      = 10.0.2.10
                  Destination = 8.8.8.8
                  Notice:

                  Source ≠ NAT instance IP
                  Destination ≠ NAT instance IP
                  
                  The NAT instance is only forwarding the traffic.
                  
                  With Source/Destination Check enabled, AWS sees that the NAT instance is neither the source nor the destination and drops the packet.
                  
                  What happens after disabling it?
                  
                  When you disable Source/Destination Check, AWS allows the NAT instance to:
                  
                  Receive packets from private subnet instances.
                  Translate the source address.
                  Forward the packets to the Internet.
                  Receive the response.
                  Send the response back to the correct private instance.


When creating a Kubernetes cluster with kubeadm on AWS EC2, you may be asked to disable Source/Destination Check on worker nodes or nodes that participate in certain CNI networking solutions because Kubernetes networking often requires a node to forward traffic for Pod IPs, not just for its own instance IP.

Why?

In Kubernetes:
        With Source/Destination Check ENABLED
        
        AWS expects the EC2 instance to communicate only for itself.
        
        ✅ Allowed:
        
        Plain Text
        1
        EC2-A (10.0.1.10) ---> EC2-B (10.0.2.10)
        Show more lines
        
        Here:
        
        Source = EC2-A
        Destination = EC2-B
        
        Both instances are sending/receiving their own traffic.
        
        ❌ Not allowed:
        
        Plain Text
        1
        Pod-A ---> Node1 ---> Node2 ---> Pod-B
        2
         
        Show more lines
        
        or
        
        Plain Text
        1
        Private EC2 ---> NAT Instance ---> Internet
        Show more lines
        
        In these cases, the intermediary instance (Node1 or NAT Instance) is trying to forward packets that belong to someone else.
        
        AWS checks:
        
        Plain Text
        1
        Is this packet's source or destination my EC2 instance?
        Show more lines
        
        If the answer is No, AWS drops the packet.
        
        With Source/Destination Check DISABLED
        
        AWS stops performing this validation.
        
        Now the EC2 instance can behave like:
        
        A router
        A NAT device
        A Firewall
        A VPN gateway
        A Kubernetes node forwarding Pod traffic
        
        Example:
        
        Plain Text
        1
        Pod-A (192.168.1.5)
        2
        |
        3
        v
        4
        Node1 EC2
        5
        |
        6
        v
        7
        Node2 EC2
        8
        |
        9
        v
        10
        Pod-B (192.168.2.7)
        Show more lines
        
        Node1 receives a packet whose:
        
        Plain Text
        1
        Source = 192.168.1.5
        2
        Destination = 192.168.2.7
        Show more lines
        
        Neither IP belongs to Node1.
        
        With source/destination check disabled, Node1 says:
        
        "I know this packet is not for me. I'll just route it to the next hop."
        
        and AWS allows it.
        
        Important Clarification
        
        Disabling source/destination check does not automatically make the EC2 a router.
        
        You still need:
        
        IP forwarding enabled in Linux
        Shell
        1
        sysctl -w net.ipv4.ip_forward=1
        Show more lines
        Proper routing tables
        CNI configuration (for Kubernetes)
        NAT rules (if acting as a NAT instance)
        
        Disabling the check simply tells AWS:
        
        "Do not drop packets just because this EC2 instance is forwarding traffic for other IPs."
        
        One-line summary
        
        Source/Destination Check Enabled: EC2 can only send/receive packets for itself.
        
        Source/Destination Check Disabled: EC2 is allowed to forward packets for other IPs (Pods, private instances, VPN clients, etc.), which is why NAT instances and some kubeadm Kubernetes networking setups require it.
        
  
