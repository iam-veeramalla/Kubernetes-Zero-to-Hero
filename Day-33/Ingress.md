 Kubernetes Ingress:
 --------------------

 1) why we are using ingress to expose our application ?

* in k8s we have services to expose our app to external world like NodePort and LoadBalncer.
* if By using this Load Balncer service type we can expose our app to external user so users will able to access the application.
* But we have few draw backs hear this Load balancer type will not support enterpress level Load balancing.
  1. Ratio based, Sticky sections, Path, Host, Domain, Whitelisting IP address and black listing IP address , TLS and web app Firewall, Ddos Attacks.
  2. if cloud Provider will not support the our K8s Cloud-controller manger it will not work.
  3. if we use LoadBalancer service type it is very costly.
  4. if we use 10 services we have to create 10 LB in cloud.


* To overcome all these issue we have ingress.
* in ingress we have 3 main things.
  1. Ingress--------------------> Yaml file.
  2. Ingress controller --------> it will read the ingress yaml files.
  3. Load Balancer -------------> it is created by the Ingress controller.
 
 Note:

       According to the ingress resource(ingress.yaml) file which we have created in our k8s. Based on that the ingress controller will create the Load balancer 
       with thet specifications mentioned in the ingress resource.


 Step:1 
 ------

 1. create deployment
 2. create a servive for deployment
 3. create a ingress resource with hostname and path, ingressClassName then only nginx ingress controller will found the ingress file and specifications.
 4. install nginx ingress controller from official website.
 5. once it was installed it fill find the valid ingressclassname in the yaml file and the ingress controller will point out to that yaml file configration.
 6. if you're using a Kubeadm cluster we have to install a cloud-controller component in our kubeadm cluster.
 7. Then only the cloud-controller manager will create a ALB/NLB in our cloud.
 8. This cloud-controller manager will watch the services type Load Balance and it will create a ALB in the cloud.
 9. if you see the external ip of the ingress controller services is in pending then the above one is the reason.
 10. the hostname was not resolving we have to add host name with nodeport to resolve the host name. becuase we changes ingress services to nodeip type from    loadbalnacer
      


 
