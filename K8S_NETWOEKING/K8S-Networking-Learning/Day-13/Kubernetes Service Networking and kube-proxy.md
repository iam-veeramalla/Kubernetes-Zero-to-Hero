Kubernetes Service Networking and kube-proxy:
=============================================

Why do we need a Kubernetes Service?
------------------------------------

* Imagine you have three backend Pods:

<img width="1013" height="367" alt="image" src="https://github.com/user-attachments/assets/d4a11b06-957e-43af-b2f4-6f2cd4420782" />

    apiVersion: v1
    kind: Service
    metadata:
      name: backend-service
    spec:
      type: ClusterIP
      selector:
        app: backend
      ports:
        - port: 80
          targetPort: 8080

* This Service provides a stable virtual IP, called a ClusterIP.

 <img width="972" height="533" alt="image" src="https://github.com/user-attachments/assets/c93f81c9-2b2d-4b56-b6a5-b96871582e91" />

How does Service traffic reach a Pod?
------------------------------------

<img width="957" height="410" alt="image" src="https://github.com/user-attachments/assets/d2e67937-ca87-421a-a861-cb5ecf174664" />

* The Service gives clients a stable destination.
* The Service networking implementation directs connections to eligible backend endpoints.

What is kube-proxy?
--------------------

* kube-proxy is a Kubernetes component that helps implement Service networking on nodes.

* It watches Service and EndpointSlice information and programs network rules so traffic to a Service can reach its backend Pods.

<img width="548" height="130" alt="image" src="https://github.com/user-attachments/assets/0b22b6ea-4f79-4b7a-a97f-cdad4940e28d" />

* kube-proxy configures the node's networking rules to direct traffic sent to the Service IP to an eligible backend.

* Important: kube-proxy is not normally sitting in the middle of every packet as a proxy process. It configures the networking rules; the Linux kernel handles packet forwarding.

* Also, some Kubernetes networking implementations, including certain CNI dataplanes, replace kube-proxy functionality.

How does kube-proxy implement Service networking?
-------------------------------------------------

Two important modes are:

<img width="976" height="532" alt="image" src="https://github.com/user-attachments/assets/b346353a-6ba5-4707-bae6-1ef3355ddd60" />

<img width="851" height="192" alt="image" src="https://github.com/user-attachments/assets/256fae62-5af6-45e9-b796-829ca64af65b" />

Practical: inspect Service networking:
---------------------------------------

Run these commands in your cluster.

<img width="535" height="673" alt="image" src="https://github.com/user-attachments/assets/e1e823b3-a19a-4574-9ad3-6e4869931ab6" />

<img width="646" height="691" alt="image" src="https://github.com/user-attachments/assets/080616a7-1181-496e-869e-e8bf3a5694c0" />

<img width="966" height="322" alt="image" src="https://github.com/user-attachments/assets/947cb053-33ad-454a-8123-23f96bf17c1f" />

Real-world troubleshooting scenario:
--------------------------------------

Your application cannot connect to a Kubernetes Service.

Follow this order:

<img width="500" height="741" alt="image" src="https://github.com/user-attachments/assets/f1f89c52-e291-41d2-9c6e-8090fa5b8868" />

<img width="970" height="501" alt="image" src="https://github.com/user-attachments/assets/529f3ec2-a6e4-4f40-a7c4-2bd7d0e27394" />
