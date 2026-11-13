1) What is Pod ?
A) in k8s we will deploy Pods not containers. Pod is just like a wrapper to container. in a pod we will
   create more than one container. it will some advantages like shared networking, storage, init containers. 

* for pod creating we have two ways
  -----------------------------------
  1. imperative by using a command
     Eg: kubectl run pod_name --image=nginx:lates
  3. declarative give specifications in a yaml file like below eg.
     Eg: apiversion, kind, metadata, spec.....

*  K8s will allocate a Cluster Ip to pods. we can access the application inside a container with this
   Pod CLUSTER iP Address.

   
First Understand the declarative manifest file in normal programing language secnario:
----------------------------------------------------------------------------------------

* if you use # before any line that line will not execute.
* spec: block is like a dictionary in yaml file.it will hold key value pairs
* if you give the values with (-) then it will consider like a list    


Now we can see how to run a Pod in imperative way:
--------------------------------------------------
kubectl run pod_name --image=image_name:tag

Now we can see how to run a Pod in declarative way:
----------------------------------------------------

In declarative we have 4 main components:
1. apiversion
2. kind
3. metadata
4. spec















   

   

   

    


