AWS VPC CNI Prefix Delegation:
==============================

* Today we'll learn how AWS VPC CNI uses IP prefixes instead of allocating individual IP addresses one at a time.

First, what is an IP prefix?
----------------------------

An IP prefix is simply a block or range of IP addresses.

For example:

    10.0.1.0/28
A /28 IPv4 prefix contains 16 IP addresses.

Instead of AWS assigning individual secondary IP addresses to an ENI, it can assign a whole /28 prefix to that ENI.

The AWS VPC CNI can then allocate individual IP addresses from that prefix to Pods.

<img width="937" height="661" alt="image" src="https://github.com/user-attachments/assets/7ce5ff88-f2c5-44d2-8ad7-a6709a9963e5" />

Remember: AWS assigns the prefix to the ENI, and the CNI manages individual Pod IP allocation from that prefix.

Why do we need prefix delegation?
---------------------------------

Let's compare the two allocation methods.

Without prefix delegation:

    - AWS VPC CNI requests individual secondary IP addresses.
    - Each individual IP uses networking address capacity on the ENI.
    - The node can reach its ENI/IP limits as the number of Pods increases.
With prefix delegation:

    - AWS assigns a block of IP addresses, such as a /28, to the ENI.
    - The CNI can allocate individual Pod IPs from that block.
    - A prefix uses one address-allocation slot on the ENI while providing 16 IP addresses.

This can significantly increase the number of Pod IPs available per node, subject to the EC2 instance's networking limits and available subnet capacity.

<img width="1028" height="243" alt="image" src="https://github.com/user-attachments/assets/26fcc9b2-83f6-4d04-bc1f-3244fba37aa3" />

How do we enable prefix delegation?

AWS VPC CNI provides this configuration:

    ENABLE_PREFIX_DELEGATION=true


When enabled, the CNI requests IPv4 prefixes from AWS instead of relying only on individual secondary IP allocations.

You may also see:

    WARM_PREFIX_TARGET=1


This tells the CNI to maintain a target of one additional warm prefix for future Pod IP allocation.

For a /28, one prefix contains 16 IPv4 addresses.

Important: WARM_PREFIX_TARGET is about keeping prefixes ready. If WARM_IP_TARGET is configured, it takes precedence over WARM_PREFIX_TARGET in determining the warm pool target.

What are the requirements?
--------------------------

Before enabling prefix delegation, check these points:

    1. Your EC2 instance type must support the required networking configuration.
    2. Your subnet must have available contiguous /28 IPv4 address blocks.
    3. Your AWS VPC CNI version must support prefix delegation.
    4. Your CNI configuration must enable prefix delegation.
    5. Your node must have enough ENI and networking capacity.

Very important: contiguous IP addresses

Imagine your subnet has 100 free IP addresses, but they are scattered across different ranges.

You might still be unable to allocate a /28 prefix because AWS needs a suitable contiguous block of 16 addresses.

So, having many free IPs does not automatically mean prefix delegation will succeed.



First, one important clarification: with prefix delegation, AWS assigns a /28 prefix to an ENI instead of assigning 16 individual secondary IP addresses. The AWS VPC CNI then gives individual IPs from that prefix to Pods.

Also, the aws-node DaemonSet runs the CNI agent on each worker node. It manages networking for the node; it does not mean that each Pod gets its own ENI.

<img width="986" height="667" alt="image" src="https://github.com/user-attachments/assets/48c8ebab-838e-483c-b669-d7cacc13362b" />

<img width="980" height="465" alt="image" src="https://github.com/user-attachments/assets/96dfd038-c6a5-473f-a780-09141719c55e" />

<img width="978" height="637" alt="image" src="https://github.com/user-attachments/assets/8502f105-918a-462a-8b1f-faf53387272a" />

<img width="982" height="837" alt="image" src="https://github.com/user-attachments/assets/b23b1813-a3aa-46bd-a1e9-b91ced0170d0" />

<img width="996" height="786" alt="image" src="https://github.com/user-attachments/assets/3781475e-92bd-4744-969e-f6ecd93ac654" />
