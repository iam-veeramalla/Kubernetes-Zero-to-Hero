Multi Container Pod's
=======================

1. we know already we can create a Pod with more that one caontainer like init / sidecar containers.

1. what init container ?
A) init container will start before the app container to perform some tasks.
   if kubelet initated a pod creation in the pod first init container will create then app container will create
   init container and app container will share the resources from the  pod.

Now we can how we can write a pod with initContainers:
-------------------------------------------------------
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
  labels:
    app: emp
spec:
  container:
    - name: app-cont
      image: nginx
      command: ['sh', '-c', 'command'] ------> in this point we are combining commands and args
      env:
        - name: "FirstName"
          value: "Aravind"
  initContainers:
    - name: init-cont
      image: nginx
      command: ['sh', '-c']
      args:['command']     ---------------> in this point we have seperated commands and arguments


* How the above manifest will work ?
  first it will create a init container and the creation of the init container will stay in pending untill the condiation which we mentioned in the init container   will satisfied.
  once the condition is satisfied inti container will come to runnig state and app conatiner will start.

  in this case our condition is to do nslookup on the service. so once the service is reachable the condition will satisfy.


Note:
-------
once you have created a pod with one init and app container. in future if you want to add another init container it will not allow you. you have to delete the
existing pod and create a new pod along with one app and two init containers.


To check the logs of a pod:
----------------------------

kubectl logs pod pod_name.


To check the container logs in a pod.
---------------------------------------

kubectl logs pod pod_name -c container_name


          
  
2. what is sidecar containers ?
A) sidecar containers  runs along with the app conatiner and provides input or take output from the app container 
