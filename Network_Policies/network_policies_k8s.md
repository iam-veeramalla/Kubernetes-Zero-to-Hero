One of the MOST important Kubernetes security features is 🔐☸️
👉 NetworkPolicy

* Because by default Kubernetes networking is very open.

* Without any NetworkPolicy:

      Pod A ───► Pod B ✅
      Pod B ───► Pod C ✅
      Pod C ───► Pod A ✅

* Every Pod can communicate with every other Pod.
Sounds convenient...

But in production, it's a security risk 🚨

💡 What is a NetworkPolicy?
A NetworkPolicy is a Kubernetes resource used to control network traffic between Pods.
Think of it as:

    🔥 Firewall for Pods
It allows you to control:

      📥 Ingress (Incoming Traffic)
      📤 Egress (Outgoing Traffic)

💡 Easy Analogy
Imagine an office building 🏢

Without security:

     🚶 Anyone can enter any room
     🚶 Anyone can leave and access any area
With security guards:

    ✅ Only authorized people can enter
    ✅ Only approved destinations can be reached
That's exactly what NetworkPolicy does.

☁️ NetworkPolicy in Amazon EKS
 
    ✅ Latest Amazon VPC CNI supports NetworkPolicies natively in EKS.

💡 Quick Revision

    🔥 NetworkPolicy → Pod Firewall
    📥 Ingress → Incoming Traffic
    📤 Egress → Outgoing Traffic
    🚫 Deny-All → Zero Trust Foundation
    🎯 podSelector → Select Pods
    🏷 namespaceSelector → Select Namespace
    ☁️ Modern EKS VPC CNI → Native NetworkPolicy Support

🔐 Best Practice → Deny All First, Then Allow Required Traffic


What is ipBlock?

* Sometimes you want to allow traffic based on an IP range instead of Pod labels.

* For example, allow traffic from:

      10.0.0.0/16

      ingress:
        - from:
            - ipBlock:
                cidr: 10.0.0.0/16
          ports:
            - protocol: TCP
              port: 8080

* This allows matching traffic from the specified IP range, subject to the network plugin's NetworkPolicy implementation.

* You can also exclude a smaller range using except:

      ipBlock:
        cidr: 10.0.0.0/16
        except:
          - 10.0.5.0/24


* This excludes 10.0.5.0/24 from the allowed CIDR

You can use:

🎯 One-Line Interview Answer
👉 NetworkPolicy is a Kubernetes resource that controls Pod-to-Pod communication by defining ingress and egress rules, enabling microservice isolation, zero-trust networking, and least-privilege security.



Now we can see about Egress in Network Policies:
------------------------------------------------

What is Egress NetworkPolicy?

* Imagine you have a backend application running inside Kubernetes.

The backend needs to communicate with:
- Database Pod.
- DNS server (CoreDNS).
- External API.

* But you don't want it communicating with every destination.

 <img width="977" height="433" alt="image" src="https://github.com/user-attachments/assets/17c9e0f0-3b7b-48e9-a46b-34c18db5554d" />

Create an Egress NetworkPolicy:
--------------------------------
Scenario:

      - Backend Pod has label app: backend.
      - Backend needs to access the database Pod.
      - Database Pod has label app: database.
      - Both are in the production namespace.
      - Database listens on TCP port 5432.
We want to allow backend Pods to communicate with database Pods on port 5432.

      apiVersion: networking.k8s.io/v1
      kind: NetworkPolicy
      metadata:
        name: backend-egress-policy
        namespace: production
      spec:
        podSelector:
          matchLabels:
            app: backend
      
        policyTypes:
          - Egress
      
        egress:
          - to:
              - podSelector:
                  matchLabels:
                    app: database
            ports:
              - protocol: TCP
                port: 5432

<img width="973" height="496" alt="image" src="https://github.com/user-attachments/assets/b471e218-d078-4445-8bcc-f07ce4095f3c" />

# Important: DNS traffic must be allowed

* This is a very important production scenario.

Suppose your backend application tries to access:

      curl http://database-service.production.svc.cluster.local:5432

* The application first needs DNS resolution to find the Service IP.

* Kubernetes DNS is normally provided by CoreDNS.

* If your egress policy blocks DNS traffic, the application might fail to resolve the Service name.
      
      egress:
        - to:
            - namespaceSelector:
                matchLabels:
                  kubernetes.io/metadata.name: kube-system
              podSelector:
                matchLabels:
                  k8s-app: kube-dns
          ports:
            - protocol: UDP
              port: 53
            - protocol: TCP
              port: 53

This permits DNS requests to matching CoreDNS Pods on UDP and TCP port 53.

Why both protocols?

      - UDP 53: Commonly used for normal DNS queries.
      - TCP 53: Used for certain DNS responses and other DNS operations, including fallback when responses are truncated.
The namespace selector and Pod selector are in the same item, so both must match.

Note: This example assumes CoreDNS Pods have the k8s-app=kube-dns label and are running in kube-system. Verify the actual labels in your cluster.

* Combine database and DNS access:
* Now let's create one complete Egress policy for backend Pods.

      apiVersion: networking.k8s.io/v1
      kind: NetworkPolicy
      metadata:
        name: backend-egress-policy
        namespace: production
      spec:
        podSelector:
          matchLabels:
            app: backend
      
        policyTypes:
          - Egress
      
        egress:
          # Allow database traffic
          - to:
              - podSelector:
                  matchLabels:
                    app: database
            ports:
              - protocol: TCP
                port: 5432
      
          # Allow DNS traffic
          - to:
              - namespaceSelector:
                  matchLabels:
                    kubernetes.io/metadata.name: kube-system
                podSelector:
                  matchLabels:
                    k8s-app: kube-dns
            ports:
              - protocol: UDP
                port: 53
              - protocol: TCP
                port: 53

Now backend Pods can:

      - Access database Pods on TCP 5432.
      - Access CoreDNS on TCP/UDP 53.
      - Not access other destinations unless another applicable policy allows them.

