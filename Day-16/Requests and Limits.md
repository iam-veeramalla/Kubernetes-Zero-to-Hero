 Requests and Limits:
 ====================

 * These request and limits are used to set the resource limits to the pods.
 * It will save other running pods which are running on the same node. 
 * If you not set any resource limits to pods then pods will consume all the resource in the node.
 * By implementing resource limits we can kill the pod with an event called OOM kill the pods which are consumming more memory.
  

 Resource Block in yaml:
 ------------------------

 resources:
   request:
     memory: ""
     cpu: ""
   limits:
     memory:
     cpu:


case: 1 
-------

* In the case one we set resource limits to min 100 mi and max 200mi.
* i will force the pod to use 150 mi by using a command and args block in the pod.
* In this case pods will use the memory with in the limit so there is no issue hear


case: 2  
-------

* In the case one we set resource limits to min 100 mi and max 200mi.
* i will force the pod to use 250 mi by using a command and args block in the pod.
* In this case pods will use the memory more than the limit so pod will be killed with OOM kill event.

case: 3  
-------

* In the case one we set resource limits to min 1000 mi and max 2000mi.
* i will force the pod to use just 150 mi by using a command and args block in the pod.
* In this case pods will not start at because we are requesting the resource more than the node resources.

  


