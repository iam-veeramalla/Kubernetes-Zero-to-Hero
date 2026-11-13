Service will below 3 things
===============================
1. Load balancing
2. service discovery
3. Exposing app to external world



Why we need service in K8S ?
------------------------------
A) if we don't have service future in k8s we have to share our pod's ip to access the application.
   but after some time the will crash for some reasons and replication set will create a new prod with other IP.
   in this case with the old pod Ip we can not access the app.

   to over come the above issue insted of accessing the application with pod ip's we will create a 
   service on top of it.

   this service acts like a loadbalancer.

   one you have created a service we will see that service like: 
  
      Eg: payments.default.svc.cluster.local


      • payments: The <service-name> defined in your Kubernetes Service manifest.
      • default: The <namespace> where the service is deployed.
      • svc: The standard Kubernetes subdomain indicating the resource is a Service.
      • cluster.local: The default cluster domain suffix (unless customized)





   by using this service IP we will access the application which are running in pods.

   once request recieaved the service then kubeproxy will distribute those requests to pods.

   Hear we have one doubt:
  ------------------------
if the new pod was created the IP will be change so service also will face same issue.how it 
will be over comed.

it will comes with Service Discovery process:
* means this services will track the applications with labeles not with IP address.
* afert a pod creash new pod will be created with new IP. but the lable will be the same for all pods.


to expose our app to outside world. we have 3 main types.
1. cluster ip: if you use this we can access app only inside the cluster
2. nodepot: if we use this we can access the app with in the organization. simply how have access to vpc.
3. load balacer: by using this we can get one elastic ip addreess. with the ip we can access it from out side.


we can expose our application in 3 ways:
------------------------------------------

1. cluster IP: we can access app inside the cluster.
2. NodePort: we can access the app with in the VPC.
3. LoadBalancer: we can access the app from the external world.

if we are using minikune for nodeport we have to do port forwording. 

Kubeshark to see the traffic flow in the our cluster



 
if you in minikube and you have exposed a servive with nord port:
--------------------------------------------------------------------

so we have to port forwording mechanisam for that we have to do port forwording:

Command: kubectl port-forwording --address 0.0.0.0 svc/service_name <local_port>:<service_port>

Eg:
Port-forward a Service

kubectl port-forward svc/python-service 8080:80
8080 → Local port on your machine
80 → Service port

Access:

http://localhost:8080
   

   
Services By piyush:
--------------------

* if you have a front-end pods, back-end pods and db.
* to expose a front-end pods to outer world we can use nodeport service.
* if you create a service with nodeport we will access the fe-pods with nodeip:nodeport we will get from the nodeport servive.
* we can alos give a specific nodeport in service of you want. so the range is b/w 30000-32767.

Nodeport-service np-svc.yaml
------------------------------

apiversion: v1
kind: Service
metadata:
  name: fe-svc
  labels:
    svc: fe-svc
spec:
  type: NodePort
  ports:
    - nodePort: 30001
      port: 80
      targetPort: 80
  selector:
    app: emp


cmd: kubectl create -f np-svc.yaml

* if you're using NodePort service in "KIND" Cluster at the time of craetiin the cluster we have to map the pots to hostmachine the contarole plane by ref kind docs.
* -containerPort: and hostPort: these two configrations we have to use in cluster yaml file unser control plane for only kind cluster.


as of now we have exposed the fe-pods with the help of nodeport

Now we can see how fe-pods will coomunucate with be-pods:
----------------------------------------------------------

* For the fe-pods and be-pods communication we will use clustserIP service.


ClusterIP-service cip-svc.yaml
------------------------------

apiversion: v1
kind: Service
metadata:
  name: cip-svc-fe-be-emp
  labels:
    svc: cip-svc
spec:
  type: ClusterIP
  ports:
      port: 80
      targetPort: 80
  selector:
    app: emp


cmd: kubectl create -f cip-svc.yaml

Endpoindts in svc:
-------------------

* Endpoints mensa terget pods IP. so all the pods ip were lister hear and automatically update hear if any of the pods will deleted and recreated.

* cmd: kubectl get ed---> to see endpoinst of the svc.

LoadBalancerIP:
----------------

* In real time we don't want to access the application with NodeportIP's.
* we need a static domine name like myapp.com
* so for thet we have to expose our service in LoadBalancer Mode.
*  it will create one external IP

  
LoadBalancer-service lb-svc.yaml
------------------------------

apiversion: v1
kind: Service
metadata:
  name: lb-svc-fe
  labels:
    svc: lb-svc
spec:
  type: LoadBalancer
  ports:
      port: 80
      targetPort: 80
  selector:
    app: emp


cmd: kubectl create -f lb-svc.yaml


ExternalName you can refer from doxs.
-------------------------------------

   
without yaml file if you want to expose a service:
--------------------------------------------------

cmd: kubeclt expose rc/deply/rs name_of_rc/rs/deply --port=80 --target-port=8081
   

    

   
  
