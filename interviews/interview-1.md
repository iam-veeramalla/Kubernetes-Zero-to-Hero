1) Diff B/W Docker and Kubernets ?

Docker:
-------
A) Docker is container service.
   Docker will not support auto healing and auto scaling.
   Docker will not support Enterprise leavel Load balancing.
   Docker will not support cluster mechanisam.




Kubernets:
----------
Kubernets is Orchisration service.
Kubernets will support auto scaling and auto healing
Kubernets will support Enterprise leavel Load balancing fro our applications with Port, Protocal, and IP Address.
Kubernets will support Cluster mechanisam.


2) what are the main componenst in Kubernets ?

A) In kubernets we have mainly Two things:
   1. Master plain
   2. Data plain

1. Master Plain:
   -------------
   1. Kubernets API
   2. ETCD
   3. Scheduler
   4. Controller manager
   5. Cloud controller manager
  
2. Data Plane:
   -----------
   1. Kube Proxy
   2. Kubelet
   3. Conatiner Runtime
  
3) What is the diff B/W docker compose and Kubernets ?

A) Kubernets is better suited fro Large scale organization.it offers more scalability, networking cababilities like Policies and third party ecosystem.


4) what is the diff B/W docker container and Kubernets Pod?

A) Docker container:
  -----------------
   we will describe all the scecicfication of container in Dockerfile. it will give much flexibility.
   

   Kubernets Pod:
   -------------
   we will describe all the specifications of conatiner in Yaml file. it will give more flexibility.
   Pod is just like a Wrapper on top of the container.
   we can run more than one container in a Pod.
   Pod in the kubernets is the Run time specification of a container.
   containers with in the Pod will share same network and volume.




5) What is name space in Kubernets ?
A) Namespace is the logical isolation of resources, netwotk, teams, Policies, RBAC,
   in our organization same k8s cluster will be used by many teams. so we can isolate the projects, environments, Teams. 



6) what is Kube Proxy In k8s ?
A) Kube-proxy works by maintaining a set of network rules on each node in the cluster,
   which are updated dynamically as services are added or removed. When a client sends a request to a service,
   the request is intercepted by kube-proxy on the node where it was received.
   Kube-proxy then looks up the destination endpoint for the service and routes the request accordingly.

   Kube-proxy is an essential component of a Kubernetes cluster, as it ensures that services can communicate with each other.




7) what are the diff types of services with in the k8s ?
A) Cluster IP
   Node Port
   Load balancer



8) What is the difference between NodePort and LoadBalancer type service?

A) When a Service is created as a NodePort type, kube-proxy updates the iptables rules with the Node IP address and the NodePort specified (or automatically assigned) in the Service configuration.
   This allows external clients to access the application using:

    <Node-IP>:<NodePort>

   Whereas, if you create a Service of type LoadBalancer, the Cloud Controller Manager (CCM) communicates with the underlying cloud provider (such as AWS, Azure, or GCP) to provision an external load balancer.
   The load balancer receives a public IP or DNS name and forwards traffic to the Kubernetes Service.

   Users can access the application using the external LoadBalancer IP or DNS name.



9) What is the role of Kubelet?

A) Kubelet manages the containers that are scheduled to run on that node. 
   It ensures that the containers are running and healthy, and that the resources they need are available.

   Kubelet communicates with the Kubernetes API Server to get information about the containers that should be running on the node, and then starts and stops
   the containers as needed to maintain the desired state. 
   It also monitors the containers to ensure that they are running correctly and restarts them if necessary.


10) Day to Day activities on Kubernets ?
    
