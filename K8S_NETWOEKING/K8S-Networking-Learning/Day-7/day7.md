AWS VPC CNI: ENI and IP Address Capacity:
=========================================

Theory — Why does ENI capacity matter?
--------------------------------------

# Remember the difference:

    - Subnet capacity: How many IP addresses remain available in the subnet.
    - Node capacity: How many ENIs and IP addresses a particular EC2 worker node can use.
    - Pod capacity: How many Pods the node can support, based on networking capacity and other Kubernetes limits.

* For example, suppose the subnet has 2,000 free IP addresses.
* Your worker node might still be unable to allocate more Pod IPs because its EC2 instance has reached its networking limits.

Example: one worker node

AWS subnet

    2,000 IPs available

EC2 worker node

ENI 1

Primary IP + secondary IP capacity

ENI 2

Additional IP capacity, if supported and attached

The EC2 instance type limits the number of ENIs and IP addresses each ENI can support.

Possible result

The subnet has free IPs, but this node cannot allocate more Pod IPs until its networking capacity or configuration issue is addressed.

One important detail: an ENI's primary IP is normally used by the node, 
while additional IPs or delegated prefixes can provide Pod networking capacity. 
Not every IP assigned to an ENI is necessarily available for a new Pod at that moment.


Practical — Inspect your EKS networking
Run these commands only if you have access to an EKS cluster and AWS CLI. They are read-only.

Step 1: List your worker nodes
kubectl get nodes -o wide


Choose a worker node and save its name.

# Step 2: Find the EC2 instance ID

    kubectl get node <node-name> \
      -o jsonpath='{.spec.providerID}'

echo


Example output:
aws:///us-east-1a/i-0123456789abcdef0


The final part is the EC2 instance ID.

# Step 3: Check the instance type

    aws ec2 describe-instances \
      --instance-ids <instance-id> \
      --query 'Reservations[].Instances[].{Type:InstanceType,Subnet:SubnetId,PrivateIP:PrivateIpAddress}'


This tells you which instance type and subnet the worker node uses.

# Step 4: Inspect its ENIs and IP addresses

    aws ec2 describe-network-interfaces \
      --filters Name=attachment.instance-id,Values=<instance-id> \
      --query 'NetworkInterfaces[].{ENI:NetworkInterfaceId,IPs:PrivateIpAddresses[].PrivateIpAddress,Status:Status}'


Check how many ENIs are attached and which IP addresses they have.

# Step 5: Check the instance's networking limits
    
    aws ec2 describe-instance-types \
      --instance-types <instance-type> \
      --query 'InstanceTypes[].NetworkInfo.{MaxENIs:MaximumNetworkInterfaces,IPv4PerENI:Ipv4AddressesPerInterface}'


This reports the instance type's documented ENI and IPv4-address limits. Prefix delegation can change how many Pod IPs you can support, so don't calculate Pod capacity from these two numbers alone.
