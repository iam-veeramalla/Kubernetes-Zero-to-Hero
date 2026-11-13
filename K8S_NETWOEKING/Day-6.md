Troubleshooting EKS Pod IP Allocation
=====================================

* Today, we'll troubleshoot a realistic AWS EKS issue: a Pod cannot start because the AWS VPC CNI cannot allocate an IP address.

Understand the scenario:
------------------------

* Imagine you deploy five replicas of an application. Three are running, but two remain in ContainerCreating.

* You check the Pod events and see an error related to CNI network setup or IP allocation.

* There are four main things to investigate:
      
      - Subnet IP address availability
      - Worker node ENI/IP capacity
      - AWS VPC CNI logs and configuration
      - AWS permissions or IP allocation errors

The goal is to identify the cause before changing anything.

Step 1 — Inspect the failing Pod
---------------------------------

Run:  kubectl get pods -n <namespace> -o wide


* Find the failing Pod and note its worker node.

Then run: kubectl describe pod <pod-name> -n <namespace>


Look at the Events section at the bottom.

    For example, you might see:
    Warning  FailedCreatePodSandBox
    Failed to setup network for sandbox:
    failed to assign an IP address to container
    

* This indicates a Pod network setup problem. It does not, by itself, prove that the subnet is exhausted.

Step 2 — Check AWS VPC CNI logs:
--------------------------------

First, locate the aws-node Pod running on the affected worker node:

    kubectl get pods -n kube-system -o wide


Then inspect its logs:

    kubectl logs -n kube-system <aws-node-pod-name> \
      -c aws-node --tail=100

Look for errors mentioning:

    - IPAM — IP address management
    - InsufficientCidrBlocks — insufficient contiguous address space for a requested allocation, where applicable
    - failed to assign IP — an IP assignment failure
    - UnauthorizedOperation — a possible AWS permissions problem

* The exact message matters. Different errors point to different causes.

Step 3 — Check subnet IP availability:
---------------------------------------

If you have AWS CLI access, run:

    aws ec2 describe-subnets \
      --subnet-ids <subnet-id> \
      --query 'Subnets[].{Subnet:SubnetId,AvailableIPs:AvailableIpAddressCount}'


Example output:

      [
        {
          "Subnet": "subnet-0123456789abcdef0",
          "AvailableIPs": 2
        }
      ]

* This means only two IPv4 addresses are currently available in that subnet.

* If availability is low, the subnet may not have enough capacity for new Pod IPs or other resources.

* Important: This command shows subnet-wide availability, not how many addresses are immediately available to a particular worker node.

Step 4 — Check worker node capacity:
-------------------------------------
Inspect the node:

    kubectl describe node <node-name>


* Check its allocated resources, conditions, and events.

* Also inspect the CNI configuration:
  
      kubectl get daemonset aws-node -n kube-system
      kubectl describe daemonset aws-node -n kube-system
      

* For detailed ENI and IP allocation information, the AWS EC2 console or AWS CLI can show the network interfaces attached to the instance and their assigned private IP addresses.

Remember:

    - A subnet can have many free IP addresses while a node has reached its ENI/IP capacity.
    - A node can have capacity while subnet exhaustion prevents new addresses from being assigned.
    - Prefix delegation changes how IP capacity is allocated, so account for it when investigating.


Three important things to remember:

1. Same subnet: In the standard AWS VPC CNI configuration, Pod IP addresses come from the VPC subnet address space associated with the node's ENIs.
2. IP allocation: AWS assigns IPs to ENIs, and the CNI configures those addresses for Pod networking. With prefix delegation, AWS can allocate blocks of addresses instead.
3. Capacity: If the subnet runs out of available IP addresses, the CNI may be unable to obtain more IP capacity, and new Pods can fail during network setup.
