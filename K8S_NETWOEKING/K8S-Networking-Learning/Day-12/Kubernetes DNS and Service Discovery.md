Kubernetes DNS and Service Discovery:
=====================================

Today's goal: Understand how one Pod finds another application using a Kubernetes Service name instead of remembering its IP address.

Why do we need Kubernetes DNS?
------------------------------

Imagine you have two Pods:

<img width="975" height="648" alt="image" src="https://github.com/user-attachments/assets/5c469d00-3d00-44cc-a149-7b382b87bfa1" />

http://backend-service:8080


Kubernetes DNS resolves the Service name to its ClusterIP.

How Kubernetes DNS works:
--------------------------

<img width="956" height="436" alt="image" src="https://github.com/user-attachments/assets/fcbb154c-6302-4c8d-81bb-09e87a61db75" />

For example:

    - Service name: backend-service
    - Namespace: production
    - ClusterIP: 10.100.20.50

The fully qualified DNS name is:

    backend-service.production.svc.cluster.local

Kubernetes DNS naming formats:
------------------------------

Kubernetes supports different DNS names depending on where the application is running.

Assume:

    - Service: backend-service
    - Namespace: production
    - Cluster domain: cluster.local (default)

<img width="995" height="537" alt="image" src="https://github.com/user-attachments/assets/062d5f32-af64-420b-9825-4b5112fa38fa" />

What is CoreDNS?:
------------------

CoreDNS

* CoreDNS is the DNS server commonly used in Kubernetes clusters. 
* It answers DNS queries for Kubernetes Services and other configured DNS records.

* When a Pod requests backend-service, its DNS resolver sends a query to the cluster DNS service.

* CoreDNS looks up the Kubernetes Service record and returns the Service's ClusterIP.

Check CoreDNS Pods:

    kubectl get pods -n kube-system -l k8s-app=kube-dns

Check the DNS Service:
  
    kubectl get svc -n kube-system kube-dns

Check CoreDNS logs:

    kubectl logs -n kube-system \
      -l k8s-app=kube-dns --tail=100
      
Practical: troubleshoot DNS resolution
--------------------------------------
* Imagine your frontend application cannot resolve backend-service.

<img width="817" height="656" alt="image" src="https://github.com/user-attachments/assets/30d45046-bf5f-4fa6-8acc-a9edfae5f079" />

<img width="745" height="553" alt="image" src="https://github.com/user-attachments/assets/9e1ecdac-38b9-4b6c-b3c1-ebfb8e79e9fb" />

<img width="750" height="542" alt="image" src="https://github.com/user-attachments/assets/71231bea-f687-4682-b139-8a63df84d74e" />

<img width="950" height="426" alt="image" src="https://github.com/user-attachments/assets/ed9a0b8b-7bcf-4ee3-9e4c-f3cb5157522e" />
