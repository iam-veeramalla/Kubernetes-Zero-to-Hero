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

