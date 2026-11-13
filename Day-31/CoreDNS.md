CoreDNS
=========

🔹 What is CoreDNS in Kubernetes?
    CoreDNS is the DNS server used inside a Kubernetes cluster.
          
          Its main job is:
          
          Convert Kubernetes service names into IP addresses.
          
          For example, instead of your application remembering:
          
          10.96.10.25
          
          it can simply use:
          
          my-service
          
          CoreDNS finds the IP address for it.

1. Why do we need DNS in Kubernetes?
   -----------
   Imagine you have:

        Frontend Pod
             │
             │ wants to communicate with
             ▼
        Backend Pod

Pods can communicate using IP addresses.

Suppose backend has:

      10.244.2.15

Frontend could call:

      http://10.244.2.15

But there is a problem.

Pod IPs are not stable.

If the backend Pod is deleted and recreated:

      Old Pod
      10.244.2.15
         ❌
      
      New Pod:
      
      10.244.3.20

Now your frontend doesn't know the new IP.

That's why Kubernetes uses Services + DNS.

2. Service gives a stable identity
   ---
Instead of connecting directly to a Pod:

    Frontend
       │
       ▼
    Pod IP

we create a Service:

    Frontend
       │
       ▼
    Backend Service
       │
       ├── Backend Pod
       ├── Backend Pod
       └── Backend Pod

The Service gets a stable ClusterIP.

For example:

      Backend Service
      ClusterIP:
      
    
      10.96.20.50

Now the frontend can use:

10.96.20.50

But Kubernetes makes this even easier.

You can use:

backend-service

CoreDNS converts:

backend-service
       ↓
10.96.20.50

3. Where is CoreDNS running?
   --------------------------
CoreDNS itself runs as Pods inside the Kubernetes cluster.

Usually you'll see it in:

kube-system

Run:

kubectl get pods -n kube-system

You'll see something similar to:

coredns-xxxxx-xxxxx
coredns-yyyyy-yyyyy

Usually Kubernetes runs more than one CoreDNS Pod for availability.

4. The basic flow
   --------------
Suppose your application wants:
       
         backend-service
The flow is:

          Application Pod
                │
                │ DNS query:
                │ "backend-service"
                ▼
          Pod DNS configuration
                │
                ▼
          CoreDNS
                │
                ▼
          Kubernetes API / Service information
                │
                ▼
          Find Service IP
                │
                ▼
          10.96.20.50
                │
                ▼
          Application

  So:

      backend-service
             ↓
          CoreDNS
             ↓
      10.96.20.50

5. Where does the Pod know CoreDNS?
   --------------------------------
This is an important part.

Inside a Pod, check:

      cat /etc/resolv.conf

You may see something like:

      nameserver 10.96.0.10
      search default.svc.cluster.local svc.cluster.local cluster.local
      options ndots:5

The important line is:

    nameserver 10.96.0.10

This is typically the ClusterIP of the Kubernetes DNS Service.


6. Kubernetes DNS Service
   ----------------------
Run:

    kubectl get svc -n kube-system

You'll typically see:

    NAME       TYPE       CLUSTER-IP
    kube-dns   ClusterIP  10.96.0.10

Even though the Service is called:

    kube-dns

the actual DNS implementation is commonly CoreDNS.

So:

      Pod
       │
       │ DNS query
       ▼
      kube-dns Service
       │
       ▼
      CoreDNS Pods

7. Why is it called kube-dns if we use CoreDNS?
   ---------------------------------------------
This confuses many beginners.

Historically Kubernetes used a DNS implementation called kube-dns.

CoreDNS later became the common/default DNS implementation.


But the Kubernetes Service is still commonly named:

kube-dns

So don't think:

    kube-dns = CoreDNS

Instead think:

      kube-dns Service
             │
             ▼
      CoreDNS Pods

8. Kubernetes Service DNS Name
   --------------------------
Suppose you have:

Service:


    backend-service


Namespace:


    default

You can access it simply using:

    backend-service

Kubernetes can resolve the fully qualified name:

      backend-service.default.svc.cluster.local

Break it down:

    backend-service
           │
           ▼
        Service
           │
           ▼
        default
           │
           ▼
          svc
           │
           ▼
    cluster.local

So:

    backend-service.default.svc.cluster.local

means:

Service backend-service in namespace default in the Kubernetes cluster.  

9. Different Namespaces
   --------------------

Suppose:

    Namespace: frontend


Service:
      
      backend-service

Actually let's say the backend Service is in:

    production

and frontend Pod is in:

    frontend

The frontend can use:

    backend-service.production

or the full DNS name:

    backend-service.production.svc.cluster.local

This is very useful in real clusters.

10. Simple Example
    -------------
Create a Service:

apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: default
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080

Kubernetes creates a Service IP:

backend
   ↓
ClusterIP
10.96.20.50

CoreDNS allows:

backend

to resolve to:

10.96.20.50

Your frontend can simply call:

http://backend

11. What happens when you use curl?
    -------------------------------
Suppose you're inside a frontend Pod:

curl http://backend

What happens?

Step 1

Application asks the operating system:

"What's the IP of backend?"
Step 2

The Pod's DNS configuration says:

Use DNS server 10.96.0.10
Step 3

DNS request goes to:

kube-dns Service
Step 4

The request reaches CoreDNS.

Pod
 ↓
kube-dns Service
 ↓
CoreDNS
Step 5

CoreDNS finds:

backend
 ↓
10.96.20.50
Step 6

The application connects:

curl
 ↓
10.96.20.50

12. CoreDNS Configuration
    ---------------------
You can inspect the CoreDNS configuration:

kubectl get configmap coredns -n kube-system -o yaml

You'll see something similar to:

    Corefile

with plugins such as:

    kubernetes
    forward
    cache
    loop
    reload
    loadbalance

Don't worry about all of these yet.

The most important one is:

    kubernetes


13. What does the kubernetes plugin do?
    -----------------------------------
    This is the plugin that allows CoreDNS to understand Kubernetes resources.

For example:

Service:


      backend

CoreDNS knows:

    backend
       ↓
    Service
       ↓
    ClusterIP

So:

    backend.default.svc.cluster.local

can be resolved.

14. What about Internet DNS?
    ----------------------

This is another important concept.

Suppose your Pod runs:

    curl https://google.com

google.com isn't a Kubernetes Service.

So CoreDNS doesn't find it from Kubernetes Services.

Instead, CoreDNS can forward the query to an upstream DNS server.


Flow:

      Pod
       │
       │ google.com
       ▼
      CoreDNS
       │
       │ not a Kubernetes Service
       ▼
      Upstream DNS
       │
       ▼
      Internet DNS
       │
       ▼
      Google IP

So CoreDNS can handle both:

    Kubernetes DNS

and:

    External DNS

15. Kubernetes DNS vs External DNS
    -------------------------------
Internal Kubernetes Service

      backend.default.svc.cluster.local
                   │
                   ▼
                CoreDNS
                   │
                   ▼
            Kubernetes Service
External website

      google.com
           │
           ▼
        CoreDNS
           │
           ▼
      Upstream DNS
           │
           ▼
      Internet DNS

16. Very Important Architecture
    ---------------------------
Remember this diagram:

                       Kubernetes Cluster
                              │
                              │
              ┌───────────────┴───────────────┐
              │                               │
           Pod A                           Pod B
              │                               │
              │ DNS query                     │
              │                               │
              └───────────────┬───────────────┘
                              ▼
                       kube-dns Service
                        ClusterIP
                       10.96.0.10
                              │
                              ▼
                         CoreDNS Pods
                        ┌───────────┐
                        │ CoreDNS   │
                        │           │
                        └───────────┘
                         │         │
             ┌───────────┘         └────────────┐
             ▼                                  ▼
      Kubernetes DNS                     External DNS
             │                                  │
             ▼                                  ▼
       Service records                    Upstream DNS



17. CoreDNS does NOT send your application traffic
     ----------------------------------------------
    This is a common misunderstanding.

CoreDNS only answers:

"What IP address belongs to this name?"

For example:

    backend.default.svc.cluster.local
                    ↓
               CoreDNS
                    ↓
                10.96.20.50

After that, CoreDNS is basically out of the way.

The actual application traffic goes:

      Frontend Pod
           │
           ▼
      Backend Service
           │
           ▼
      Backend Pod

Not:

      Frontend
       ↓
      CoreDNS
       ↓
      Backend
  
That's important.

18. CoreDNS vs kube-proxy
     ---------------------
    Another common confusion:

CoreDNS

Responsible for:

Name → IP

Example:

backend
 ↓
10.96.20.50
kube-proxy

Helps implement Service networking and forwarding/load-balancing behavior for Service traffic, depending on the cluster networking implementation.

Simplified:

CoreDNS
backend
   ↓
10.96.20.50




kube-proxy / dataplane
10.96.20.50
   ↓
Backend Pod

🧪 Hands-on — Let's Test CoreDNS
 ---------------------------------------
Create a temporary Pod:

kubectl run dns-test \
  --image=busybox:1.36 \
  --restart=Never \
  -it \
  --rm \
  -- sh

Inside the Pod:

cat /etc/resolv.conf

You should see the Kubernetes DNS server.

Then:

nslookup kubernetes.default

You should get an IP.

Try:

nslookup kube-dns.kube-system.svc.cluster.local

Then try:

nslookup google.com

Now you've tested both:

Kubernetes DNS

and:

External DNS
🧠 The easiest way to remember CoreDNS

Think of CoreDNS as the phone directory of your Kubernetes cluster.

Instead of remembering:

10.96.20.50

your application can remember:

backend

CoreDNS says:

backend
   ↓
10.96.20.50

Then the application communicates with that IP.

One-line interview answer

CoreDNS is the DNS server running inside Kubernetes. It resolves Kubernetes Service names such as backend.default.svc.cluster.local to their corresponding Service IPs and can forward external DNS queries to upstream DNS servers.

If you understand the flow:

Pod
 ↓
/etc/resolv.conf
 ↓
kube-dns Service
 ↓
CoreDNS
 ↓
Service IP
 ↓
Application traffic

then you understand the core concept of CoreDNS.
