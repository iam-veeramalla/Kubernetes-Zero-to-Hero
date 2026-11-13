NameSpaces: 
=============

In k8s cluster if you want to isolate the objects/ resources like pods, services etc.. then we have to use Namespaces.


By default we will get:
1. default ns
2. kube-system ns
3. kube-public

we can create a sepearte ns for our environments.

* resources with in the namespace will communicate with each other.
* if resource need to communicate with other ns resources then we required FQDN(fully qualified resource name).

Now we can see how to create a ns in Declarative way:
-----------------------------------------------------
apiVersion: v1
kind: Namespace
metadata:
  name: demo

cmd: kubectl create -f ns.yaml


Now we can see how to create a ns in imparative way:
----------------------------------------------------

cmd: kubectl create ns demo

To delete a ns:
---------------

cmd: kubectl delete ns ns_name


Scenario:
---------
prerequisites:

1. 2 diff namespaces eg: (demo, default)
2. create a deplyment in both the ns.
3. expose the deployemnts with clusterIP on both the ns.

step:1 
------

1. Get the pods from ns.--> kubectl get pods -n demo
2. login to the pod:---> kubectl exec -it pod_name -- sh -ns=demo
3. after login ping/curl to default pod IP from this demo ns pod: curl default_ns_podIP
4. we will see they are communicating.
5. because IP address is cluster wide.
6. we access with IP adress from any ware in the cluster.



step:2 
------

 * if you want to communicate with with service_name it will broke the connection becuase Service names are namesapce scpoed. if one pod in demo ns want to
    communicate with a service which is running on default ns for thet we have to use (FQDN) like service_name.ns.svc.cluster.local.


* But with in the same namesapce we can call it with servicename.


 







