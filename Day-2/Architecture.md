Kubernets Architecture divide into 2 parts:
============================================

1. Control Plain components
2. Data Palin Componenets

* Before going to components concept we have understand one thing if you want to Run a container we need a container Run time without this run time our container will not Run.

* In Docker we have Docker shim as Container run time.

* Kubernetes is advanced than Docker it will support most of the advanced Enterprise level concepts like Auto scaling, Auto healing, load balancing, clustering with master and worker nodes.

* in production we always multi master and multi worker structure.

* in k8s our request will not directly go to worker node(Data plane). it will go through master node(control plane).

* in k8s the lowest level of deployment is POD. Pod is just like a wraper to our container with advances features.


Control plain components:
-------------------------


API-server:
----------

* The Kubernetes API Server (kube-apiserver) is the central brain and the only entry point to a Kubernetes cluster.

      * it will acts as a core component in k8s.
      * it will decide pod creation.
      * it will take all incomming requests from out side from the cluster.
      * it will expose k8s to external world.
     
      * Eg: user is trying to create a Pod. he reaches API server with a command so this API server will decide which worker node is free to deploy this Pod. But to schedule that Pod on thet worker node we will use Scheduler

The 4 Stages of an API Request:

* When you run a command like kubectl apply -f deployment.yaml, the request doesn't just instantly create pods. It must successfully pass through four strict checkpoints inside the API Server:

        [ Request ] ──> ( 1. Authentication ) ──> ( 2. Authorization ) ──> ( 3. Admission Control ) ──> [ Saved to etcd ]

      1. Authentication (Who are you?): The API Server checks your identity using certificates, tokens, or OIDC (like AWS IAM if you are using EKS). If it can't verify who you are, it throws a 401 Unauthorized error.
         
      2. Authorization (What are you allowed to do?): It checks your RBAC (Role-Based Access Control) permissions. Do you have the right to create a deployment in this specific namespace? If not, it throws a 403 Forbidden error.
      
      3. Admission Control (Is this request safe/valid?): This is where specialized plug-ins check or modify the request. For example, a Mutating Webhook might automatically inject an Istio Envoy sidecar container into your pod, and a Validating Webhook will block the pod if it asks for more memory than your cluster allows.
      
      4. Storage (Save to etcd): Once the request passes all validation steps, the API Server converts the YAML data into JSON and stores it securely in etcd.


Crucial Architectural Concepts:

It is Stateless: 

      The API Server does not store any data locally on its hard drive. Because it is completely stateless, you can easily scale it horizontally. In a production multi-AZ cluster, AWS EKS automatically deploys multiple copies of the API Server behind a Network Load Balancer (NLB) to ensure high availability.


It Uses a Declarative Model (The Controller Pattern)

      The API Server doesn't actually create your pods or configure your AWS ENIs. It simply stores your Desired State (e.g., "I want 3 replicas of this billing app").
      Other background workers called Controllers (like the kube-controller-manager or the aws-node daemonset) constantly watch the API Server to see what the desired state is, compare it to the Current State on the worker nodes, and make changes to match them.      


The Kubelet Connection

      Every single worker node runs a tiny agent called the Kubelet. The Kubelet establishes a continuous secure stream (using HTTP/2 web sockets) directly to the API Server. It watches for any pods assigned to its specific node, spins them up, and continuously reports back the health of those containers to the API Server.
        
  
Scheduler:
---------
       
* it will responsible for scheduling a Pods and resources in K8S.
* The Kubernetes Scheduler is a control-plane component that decides which worker node should run a newly created Pod.

<img width="972" height="712" alt="image" src="https://github.com/user-attachments/assets/14c85cf5-ad4a-4362-a528-87892c6bcaf7" />

* How does the scheduler choose a node?

It checks several things:

      CPU and memory: Does the node have enough available resources for the Pod's requests?
      Taints and tolerations: Is the Pod allowed to run on that node?
      Node selectors and affinity: Does the node match the Pod's placement rules?
      Pod affinity and anti-affinity: Should the Pod run near or away from other Pods?
      Topology spread constraints: Should Pods be distributed across nodes or availability zones?

* The scheduler filters out unsuitable nodes, then scores the remaining candidates and selects a node.

<img width="982" height="737" alt="image" src="https://github.com/user-attachments/assets/071bb182-819d-4324-b42e-1f4ba7a28896" />



ECTD:
-----
  
* It will acts like DB of our K8s cluster.
* it will store in Key=value pair.

What is etcd?

      Think of etcd as the database of Kubernetes.

Kubernetes stores its cluster state in etcd, including:

      - Pod information
      - Deployment configurations
      - Services
      - ConfigMaps
      - Secrets
      - Node information
      - Cluster configuration and state

How etcd works:

<img width="961" height="362" alt="image" src="https://github.com/user-attachments/assets/8d96db70-b529-40e5-b96c-548100494d3c" />

For example, when you execute:

      kubectl create deployment nginx --image=nginx


* The API server processes the request and Kubernetes persists the desired state in etcd.
* The controllers then work to make the actual cluster state match the desired state.

<img width="887" height="620" alt="image" src="https://github.com/user-attachments/assets/4a609773-e7bf-42d6-9356-4648923a04ab" />

What happens if etcd has a problem?

* Imagine your EKS etcd database becomes unavailable.

Possible consequences:

      1. kubectl get pods may fail because the API server cannot retrieve cluster state.
      2. You may not be able to create new Deployments.
      3. You may not be able to scale applications.
      4. Kubernetes may not be able to persist updated resource states.


Will existing Pods immediately stop running?

      No. Existing workloads can continue running on worker nodes for some time because the containers are already running.

However, Kubernetes management and reconciliation can be affected.

This is an important production interview question.

How do we monitor etcd in EKS?

As a DevOps engineer, you should understand these metrics:

<img width="996" height="346" alt="image" src="https://github.com/user-attachments/assets/e4e8689c-2b42-4fa5-92d7-a2fb26ab6c1d" />






     4. Controller Manager:
        ------------------
        * Controller Manager is the brain that watches and fixes things inside the cluster.

        * It runs many controllers.

        📌 Think of it like a supervisor 👨‍💼

            Watches everything

            Takes action when something is wrong

      🔁 What does Controller Manager do?

          It continuously:

           Checks current state

           Compares with desired state

      Takes action to fix it
   🧩 Important Controllers inside Controller Manager
  🔹 Node Controller

       Checks node health

       If a node goes down → marks it NotReady

  🔹 ReplicaSet Controller

      Ensures correct number of pods

      Example:

      Desired: 3 pods

      One pod crashes → creates a new pod

   🔹 Deployment Controller

        Handles rolling updates

        Ensures zero downtime deployments

  🔹 Job / CronJob Controller

      Runs jobs

  5. Cloud Controller Manager
     ------------------------

     Cloud Controller Manager handles cloud-specific tasks only.

     It talks to:

         Cloud APIs

         Load balancers

         Volumes

          Cloud networking

  🧩 What Cloud Controller Manager controls?
      🔹 Node Controller (Cloud version)

           Detects when cloud VM is deleted

           Removes node from cluster

  🔹 Service Controller

       Creates cloud Load Balancer

   Example:

       type: LoadBalancer


  Automatically creates AWS ELB / Azure LB / GCP LB

   🔹 Route Controller

        Manages cloud routing rules

   🔹 Volume Controller

       Attaches / detaches cloud disks

       Example: EBS, Azure Disk, Persistent Disk

   🧠 Easy way to remember

   🧠 Controller Manager

    “Fixes Kubernetes things”

   ☁️ Cloud Controller Manager

       “Talks to the cloud”

   🏢 Enterprise understanding (simple)

      Controller Manager = core Kubernetes logic

      Cloud Controller Manager = cloud integration layer

  Separation improves:

    Portability

     Security

    Cloud neutrality


  🏁 One-line summary

     👉 Controller Manager keeps the cluster healthy
     👉 Cloud Controller Manager connects Kubernetes to the cloud



    
2 . Data plain components:
   ----------------------
   
Kubelet:
--------
      * Kubelet is an agent that runs on every Kubernetes worker node and ensures that the containers described in the Pod specifications assigned to that node are running and healthy.
      * Kubelet: Makes sure the assigned Pod actually runs on its node.
      * For example, suppose you deploy your Empower application with three replicas.
      * Kubernetes schedules those Pods onto worker nodes, and the kubelet on each selected node works to make its assigned Pods run according to their specifications.
      * It observes the Pod specification assigned to its node and coordinates with the container runtime through the Container Runtime Interface (CRI).
      * kubelet is responsible component for maintianing the deployed pod in the worker node.
      * it will check the Pod is running or not. if it is not running it will tells K8s control plain.

Kubelet checks the Pod configuration and starts containers:

      You submit a Pod YAML to the Kubernetes API server.
      The scheduler assigns the Pod to a worker node.
      Kubelet on that node observes the assigned Pod specification through the API server.
      Kubelet coordinates with the container runtime to create and start the containers.
      Kubelet monitors container health and reports Pod status.
      Kubelet doesn't independently schedule the Pod or decide which node should run it.

Kubelet monitors Pod health

      Kubelet performs configured health probes and takes the appropriate action.
      - Liveness probe: Determines whether a container should be restarted.
      - Readiness probe: Determines whether a container is ready to receive normal Service traffic.
      - Startup probe: Allows a slow-starting container time to initialize.
      For example, if your Empower application fails its liveness probe repeatedly, kubelet can restart its container according to the configured probe and restart policy.



Container Runtime:
------------------
       * in k8s we will use "Docker shim", Container D, Cri-O or any other container runtimes which implement k8s container interface.
         
Kube-Proxy:
-----------

      * it will provide networking, IP address to pods and load balacing in k8s. it will usese Ip tables in linux machine.
      * kube-proxy acts as a data-plane controller that writes the routing rules on each node so that the Linux kernel handles the actual traffic routing.
      * kube-proxy runs on every single node in your cluster. It constantly watches the Kubernetes API server for two things:
        
               1. When a new Service is created (and assigned a ClusterIP).
               2. When Pods (Endpoints) are added, removed, or change health status.
  
* The Rule Maker (Data Plane Role)
  
          When kube-proxy detects a change, it updates the low-level networking rules on its host node. Depending on your cluster configuration, it uses either:
         • IPVS (IP Virtual Server): L4 load balancing built into the Linux kernel (standard for modern/large clusters).
         • iptables: A Linux packet filtering system.

* It translates a high-level concept like "Send traffic for 10.96.0.1 to these 3 pods" into low-level firewall/routing rules.

* The Traffic Flow (What happens to your request)

         Because kube-proxy already configured the node's kernel rules ahead of time:
         1. Your application sends a packet to the Service IP (payments.default.svc.cluster.local).
         2. The packet hits the Linux kernel of the host node.
         3. The kernel looks at the iptables or IPVS rules created by kube-proxy.
         4. The kernel intercepts the packet, changes the destination IP from the Service IP to one of the healthy Pod IPs (performing DNAT / Destination Network               Address Translation), and load-balances the request.
         5. The packet goes directly to the Pod
               

   
      
Scenario 2: Pod-to-Service Communication
            
When a Pod behind Service 1 tries to send a request to payments.default.svc.cluster.local, the traffic never leaves the Kubernetes network engine. It              all happens at the Linux kernel level:

            1. The Request Leaves Pod 1: Pod 1 sends an HTTP request addressed to the Service 2 ClusterIP (e.g., 10.96.0.1).
            2. Hitting the Host Kernel: The packet leaves the Pod's virtual network interface and enters the host node's Linux kernel.
            3. The Kernel Inspection: The kernel checks its iptables or IPVS rules (which kube-proxy previously wrote). It sees a rule that says: "If traffic is                  destined for 10.96.0.1, intercept it."
            4. DNAT (Destination Network Address Translation): The kernel randomly selects one of the healthy backend Pod IPs belonging to Service 2 (e.g.,                       192.168.1.45) based on its load-balancing algorithms. It alters the packet header, changing the destination from the Service IP to the specific                    Pod IP.
            5. Direct Delivery: The packet is then routed across the cluster network directly to the node hosting that specific Service 2 Pod.
            

Actual Flow:

   
      1. The Request Hits the Kernel: The packet arrives, addressed to the virtual Service IP.
      2. Consulting the Rules: The kernel checks the iptables or IPVS rules (programmed by kube-proxy).
      3. Choosing the Pod (Load Balancing): The rules tell the kernel which Pod IPs are currently healthy. The kernel randomly (or via round-robin) picks the best           target Pod IP.
      4. Performing DNAT: Now the kernel rewrites the packet header, swapping the virtual Service IP out and putting the chosen Pod IP in.
      5. Routing the Traffic: The kernel routes the packet directly to its final destination.



CoreDNS
---------

* The Watcher (Control Plane)

         Just like kube-proxy, CoreDNS constantly watches the Kubernetes API server.
         • Whenever a new Service is created, CoreDNS automatically generates a DNS record for it.
         • It follows a strict naming convention: <service-name>.<namespace>.svc.<cluster-domain>.

* The Resolution Flow (The Step-by-Step)

         When Service 1 wants to talk to Service 2, CoreDNS steps in before any network packets are even routed:
         1. The Question: Inside Pod 1, your application code tries to connect to http://cluster.local.
         2. The Phone Call: The Pod's operating system sends a standard DNS query to the cluster's central DNS server, which is CoreDNS (running as a set of                    standard pods in the kube-system namespace).
         3. The Answer: CoreDNS looks at its internal database, finds the record for payments.default, and replies back to the Pod with the Service ClusterIP                   (e.g., 10.96.0.1).
         4. The Hand-off: Now that Pod 1 has the IP address, it can finally create the actual data packet.




      
