K8S-ingress
============


why we need ingress ?

service is offering

1. Discovery
2. Load balancing
3. Exposing the app to outer world

this services offering load balancing it works with round robin mechanism.

this offering very simple LB. But when we compared with the Enterprise Load Balancing that offer Ratio based, Sticky sections, Path, Host, Domain, Whitelisting and black listing, TLS and Firewall.


But k8s Service LB not offering these many Enterprises stranded things.

for our application we have 10 micro services and for all 10 services we crated 10 services with LB type in cloud provider 10 static Public ip address will be crated so it will charge more for 10 static ip address.



to overcome all the issues K8S comes with Ingress
                                          --------

K8s Will not directly support this ingress it need help from ingress controller this ingress controller will be provided by other companies like Nginx, F5, Ambassador, HAproxy.


in K8S we will write ingress manifest file with set of rules to implement this we need to deploy controllers in our K8S.

What is this ingress controller ?

it is Load Balancer + API Gateway which provides Enterprise standards in K8S. 



some ingress controllers sit inside a cluster inside like (nginx) we have to deploy them like a PODS.

but few ingress controller sit outside the cluster like (F5) and send the traffic via VXLAN tunnels.



Now i want implement ingress controller on the minikube.
==========================================================

Step:1 in kubernets cluster create an ingress.yaml with "host", "service name" and "service IP"

step:2 open the url https://v1-32.docs.kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/ ---> officail documentation provided by k8s.

Step:3 follow the steps and enable the ingress.

check the ingress pods are running or not for that:
---------------------------------------------------

Command: kubectl get pods -n ingress_namespace

if you want to check the ingress.yaml file is identified by ingress controller or not:
-----------------------------------------------------------------------------------

Command: kubectl logs ingress_pod -n ingress_namespace

step:4 In minikube additionally we have to one more setup:
----------------------------------------------------------
why we are doing this because this domine is not purchased any domine providers. it a dummy domins this domine have no ip so we are adding this bummy host name with ingress address ip in the 
we have to update /ect/hosts---> vim /etc/hosts--> Ingress_address Hostname





 
