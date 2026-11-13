First, why do we need Calico?

* Imagine you created a Kubernetes cluster using kubeadm:

      - One control-plane node.
      - Two worker nodes.
      - Multiple Pods running on different nodes.

<img width="760" height="261" alt="image" src="https://github.com/user-attachments/assets/bf6e9af6-81fd-4fa9-867e-757746ef203e" />

How will the frontend Pod on Worker 1 communicate with the database Pod on Worker 2?

* Kubernetes needs a networking solution to provide Pod-to-Pod connectivity.

* That's where Calico comes in.

* Calico provides Pod networking and can enforce network policies.

<img width="812" height="341" alt="image" src="https://github.com/user-attachments/assets/a1210ef4-0632-4d7b-bedc-d7870bf9d8f8" />

How does Calico work when a Pod is created?
---------------------------------------------

* Let's understand the actual flow.

<img width="927" height="705" alt="image" src="https://github.com/user-attachments/assets/e695e132-5d5d-4e3a-a40d-e6fb7121ebf8" />

<img width="537" height="316" alt="image" src="https://github.com/user-attachments/assets/4b56f4f5-89d1-4bb8-9fe9-aa7e220c74f7" />

How does Pod-to-Pod communication work across nodes?
---------------------------------------------------

<img width="952" height="528" alt="image" src="https://github.com/user-attachments/assets/ba8568cf-9ccf-4115-ae6f-6d8462ccd799" />

<img width="525" height="535" alt="image" src="https://github.com/user-attachments/assets/d18e64da-4b36-4201-bf30-61ee7705d0ec" />

How does traffic reach the other node?
---------------------------------------

Calico supports different networking modes. Let's understand the two important ones.

<img width="708" height="301" alt="image" src="https://github.com/user-attachments/assets/0609ae55-c396-4673-bd89-b615a0186cd9" />
<img width="961" height="662" alt="image" src="https://github.com/user-attachments/assets/cba263c8-4157-49c0-8e19-37260352cdd0" />

<img width="937" height="390" alt="image" src="https://github.com/user-attachments/assets/7dfe488c-d683-4275-87a8-766a62d4f84b" />

<img width="767" height="536" alt="image" src="https://github.com/user-attachments/assets/ac531a9e-50b5-4746-8f59-4324d67be77e" />


Enforcing Netwoking policies with Caloco CNI:
-----------------------------------------------

Yes. Calico VXLAN and Kubernetes NetworkPolicies work together. VXLAN handles pod-to-pod packet transport between nodes, while Calico enforces network policies to control which pods can communicate.

Let's take your kubeadm cluster as an example.

Understand the architecture
----------------------------

Suppose you have two worker nodes:

Worker-1: Frontend Pod
Worker-2: Backend Pod
Calico CNI: Provides pod networking.
VXLAN: Encapsulates pod traffic between nodes.
Calico NetworkPolicy: Controls whether communication is allowed or denied.


<img width="722" height="317" alt="image" src="https://github.com/user-attachments/assets/f7cf9cf6-751c-4756-9fec-86478cc80b02" />

# How does Calico enforce NetworkPolicies?
Consider this scenario:

You have three pods:

      - Frontend
      - Backend
      - Database
Your requirement:

      - Frontend can communicate with Backend.
      - Backend can communicate with Database.
      - Frontend cannot communicate directly with Database.
      - Other unauthorized traffic must be blocked.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/39da7fdd-5793-4123-b419-489753743940" />

Calico enforces these rules at the workload network interfaces.

In the standard Linux iptables dataplane, Calico programs packet-filtering rules on the worker nodes.

The packet flow is conceptually:

      1. Application generates a network packet.
      2. Calico evaluates applicable network policies.
      3. If allowed, the packet continues through the networking stack.
      4. If it needs to reach another node, VXLAN encapsulates it.
      5. The receiving node decapsulates the packet.
      6. Calico applies the destination workload's applicable policies.
      7. If allowed, traffic reaches the destination pod.

The exact processing hooks depend on the configured dataplane.

Practical example: Enforce NetworkPolicy
Suppose your Backend Pod listens on port 8080.

You want only Frontend Pods to access it.

First, label your pods:

      kubectl label pod frontend app=frontend
      kubectl label pod backend app=backend

Create a NetworkPolicy:


            apiVersion: networking.k8s.io/v1
            kind: NetworkPolicy
            metadata:
              name: allow-frontend-to-backend
              namespace: default
            
            spec:
              podSelector:
                matchLabels:
                  app: backend
            
              policyTypes:
                - Ingress
            
              ingress:
                - from:
                    - podSelector:
                        matchLabels:
                          app: frontend
            
                  ports:
                    - protocol: TCP
                      port: 8080

Apply it:
kubectl apply -f network-policy.yaml


<img width="1001" height="350" alt="image" src="https://github.com/user-attachments/assets/c128465c-ecbf-4b63-a650-e1409e9df32a" />

The last point is important: this policy only selects Backend Pods. It does not automatically block Frontend-to-Database traffic.

How to troubleshoot NetworkPolicies

Useful commands:

            # List network policies
            kubectl get networkpolicy -A
            
            # Describe a policy
            kubectl describe networkpolicy allow-frontend-to-backend
            
            # Check Calico nodes
            kubectl get nodes.crd.projectcalico.org
            
            # Check Calico pods
            kubectl get pods -n calico-system
            
            # Check VXLAN configuration
            kubectl get ippools.crd.projectcalico.org -o yaml


For Calico-specific policies, you can also create GlobalNetworkPolicy resources, which can apply across namespaces.


Interview question
Imagine this production scenario:

You have a kubeadm cluster with Calico VXLAN.

      - Frontend Pod runs on Worker-1.
      - Backend Pod runs on Worker-2.
      - VXLAN tunnel is working.
      - Frontend cannot access Backend.
      - Pod IP connectivity was working before you applied NetworkPolicy.

What would you check first?

My expected troubleshooting areas:

      1. NetworkPolicy selectors.
      2. Source and destination pod labels.
      3. Ingress and egress policy rules.
      4. Destination port and protocol.
      5. Calico policy enforcement and node logs.

GlobalNetworkPolicy (GNP):
--------------------------


GlobalNetworkPolicy (GNP) is a Calico-specific network policy that allows you to enforce network security rules across multiple namespaces in a Kubernetes cluster.

Think of it this way:

      - NetworkPolicy → Kubernetes standard policy, scoped to a namespace.
      - GlobalNetworkPolicy → Calico policy, can apply across the entire cluster or selected workloads across namespaces.

Real-world example

Suppose your kubeadm cluster has three namespaces:

      Kubernetes Cluster
      │
      ├── dev namespace
      │    ├── frontend
      │    └── backend
      │
      ├── staging namespace
      │    ├── frontend
      │    └── backend
      │
      └── production namespace
           ├── frontend
           └── backend


Your security requirement:

Block SSH traffic (port 22) to all application pods across all namespaces.

Instead of creating separate NetworkPolicies in every namespace, you can use a Calico GlobalNetworkPolicy.


GlobalNetworkPolicy YAML example:


apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy

metadata:
  name: block-ssh-globally

spec:
  selector: all()

  types:
    - Ingress

  ingress:
    - action: Deny
      protocol: TCP
      destination:
        ports:
          - 22

<img width="1000" height="365" alt="image" src="https://github.com/user-attachments/assets/9aef58d3-958a-4ca2-b2fb-49483ce4bd61" />

<img width="963" height="387" alt="image" src="https://github.com/user-attachments/assets/4797906b-0bcb-4cc9-9568-f44c1968ee69" />

<img width="1006" height="775" alt="image" src="https://github.com/user-attachments/assets/aedc810e-98dc-4206-aa53-425e43909591" />







