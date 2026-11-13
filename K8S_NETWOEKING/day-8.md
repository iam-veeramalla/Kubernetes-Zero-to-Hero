Day 8 — AWS VPC CNI Warm IP Pool
=================================

* Let's understand the entire Day 8 in simple statements, step by step, with examples.

1. What is a Warm IP Pool?

* In AWS EKS, every Pod needs an IP address to communicate with other Pods and services.

* The AWS VPC CNI can allocate some IP addresses in advance so they are ready when new Pods are created.

* These pre-allocated, unused IP addresses are called the warm IP pool.

Example:

    Imagine your worker node has 3 running Pods and 2 extra IP addresses ready for new Pods.


<img width="1102" height="693" alt="image" src="https://github.com/user-attachments/assets/695137ee-559e-4289-866c-dd76d55395de" />
<img width="1001" height="387" alt="image" src="https://github.com/user-attachments/assets/8d623342-df6e-454f-bcc6-0350e37b4089" />
<img width="995" height="858" alt="image" src="https://github.com/user-attachments/assets/d0fcc1db-8bff-4724-a2ca-947b59c10e56" />
