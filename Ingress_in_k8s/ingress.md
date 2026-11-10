Kubernetes Ingress and Ingress Controller:
-------------------------------------------

Today's goal: Understand how external users access multiple applications in an EKS cluster using one entry point, 
and how an Ingress Controller routes requests to the correct Kubernetes Services.

* We'll cover Ingress, Ingress Controller, AWS Load Balancer Controller, traffic flow, and practical troubleshooting.

<img width="1000" height="770" alt="image" src="https://github.com/user-attachments/assets/a23ef2ae-4ff3-4792-a668-a8dd6f98a7ce" />

2. What is Ingress?

* Ingress is a Kubernetes API resource that defines rules for routing external HTTP/HTTPS traffic to Services inside the cluster.

For example:
- / routes to frontend.
- /api routes to backend.
- /admin routes to admin.
Think of Ingress as a traffic-routing instruction.

<img width="987" height="437" alt="image" src="https://github.com/user-attachments/assets/0e151b35-d650-4699-82ea-5c28944c117b" />

3. What is an Ingress Controller?
   
* This is a very important concept.

<img width="925" height="512" alt="image" src="https://github.com/user-attachments/assets/4a37e50d-c7d5-472f-b1ad-ba50dbcd83b2" />

4. AWS EKS — How does Ingress work?

In EKS, we'll focus on the AWS Load Balancer Controller.

AWS Load Balancer Controller:

It watches Kubernetes Ingress and Service resources and manages the corresponding AWS load balancers.

When you create an Ingress resource configured for the AWS controller:

    The controller detects the Ingress resource.
    It provisions an AWS Application Load Balancer (ALB).
    It creates target groups for the backend Services.
    It configures listeners and routing rules.
    Incoming traffic is directed to the appropriate application.

Traffic flow in EKS

    For an ALB using IP target mode:

<img width="941" height="482" alt="image" src="https://github.com/user-attachments/assets/0a8be6e9-4737-4f76-a8d5-83a89e2407fc" />

Important distinction:

      - Instance target mode: ALB sends traffic to worker nodes through NodePort, then Kubernetes Service networking forwards it to a Pod.
      - IP target mode: ALB sends traffic directly to registered Pod IPs.

For EKS with AWS VPC CNI, IP target mode is commonly recommended because it avoids the extra NodePort and Service forwarding hops

# Practical — Create an Ingress YAML

Let's assume you already have:

    - Namespace: production
    - Service: backend-service
    - Service port: 80
    - Backend Pod application port: 8080

Create an Ingress:

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: backend-ingress
      namespace: production
      annotations:
        alb.ingress.kubernetes.io/scheme: internet-facing
        alb.ingress.kubernetes.io/target-type: ip
    spec:
      ingressClassName: alb
      rules:
        - host: example.com
          http:
            paths:
              - path: /api
                pathType: Prefix
                backend:
                  service:
                    name: backend-service
                    port:
                      number: 80

Let's understand the important fields.

<img width="927" height="496" alt="image" src="https://github.com/user-attachments/assets/4bc90bc8-5158-4f9a-ace8-c87bf12b53e8" />

<img width="847" height="846" alt="image" src="https://github.com/user-attachments/assets/f1d270fa-1288-4b40-8740-c17eed6f71a3" />

If the controller is installed in another namespace, check that namespace instead.

<img width="912" height="437" alt="image" src="https://github.com/user-attachments/assets/203d488f-8852-44e9-9b6d-1dc529a0cbc1" />

Real-world troubleshooting:

Suppose your Ingress is created, but you cannot access the application.

Follow this order:

Step 1 — Check Ingress

    kubectl get ingress -A

Step 2 — Check Ingress events

    kubectl describe ingress backend-ingress -n production

Step 3 — Check the Service

    kubectl get svc -n production

Step 4 — Check EndpointSlices

    kubectl get endpointslice -n production
    
Step 5 — Check controller logs

    kubectl logs -n kube-system \
      deployment/aws-load-balancer-controller --tail=100

<img width="901" height="632" alt="image" src="https://github.com/user-attachments/assets/f32b9c1d-19be-4657-bee8-281935633999" />

