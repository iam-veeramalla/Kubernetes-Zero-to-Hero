Health Probes
==============

Probes:
-------
means some thing is monitoring, inspecting our system.

Health Probes in k8s:
---------------------
* In k8s we have mainly 3 Probes.
  1. LivenessProbe
  2. RedinessProbe
  3. StartupProbe

1. LivenessProbe:
   --------------
   * it will monitore the application if any failure happen on the application end or intermediate network issue then this Probe will takecare
     of restarting the pods and make them available.

2. RedinessProbe:
   ---------------
   * This Probe will take care of the application is ready to accept the traffic or not. if the application is still not ready for accepting the traffic this
     probe will remove that pod from loadbalacing. once the app was ready then this probe will re-add that pod in loadbalacing to accept the traffic.

  3. StartupProbe:
     --------------
     * it is mainly used fro slow and legacy application. these applicatins like java will take some time to start the application.
     * in this case if the above probes start there work before stating the application it will causes the application pods creashes.
     * so by using this startup probe we will tell once the applicartion started then only above probes will start there work.


  * this probes will support 3 types of protocols.
    1. HTTP
    2. TCP
    3. Command
   
  * These Probes are part of containers.


LivenessProbe block in yaml:
----------------------------

1. if you are using a command

        livenssProbe:
          exec:
            command:
              - cat 
              - /tmp/healthy
          initialDelaySeconds: 5
          periodicSeconds: 5 

* in this case by using a command to check from the end point we are getting the responce or not. if you delete the end point folder then this probe
  will not get any responce then probe will fail and start resating the container/pod.



2. if you are using a HTTP protocol:
      
        livenssProbe:
          httpGet:
            command:
              path: /health 
              port: 8000
          initialDelaySeconds: 5
          periodicSeconds: 5 


3. if you are using a TCP protocol:

      livenssProbe:
        tcpSocket:
            port: 8000
        initialDelaySeconds: 15
        periodicSeconds: 5 


Main fields in these Probes:
----------------------------

    InitialdealySeconds: it will tell when first check will start eg: after 15 sec.**
    periodicSeconds: it will tell after first check when again the check need to perfrom eg: 5 sec after first check every 5 sec it will do checks.
    successThreshold: it will tell if the check is success for 2 times then it will mark as healthy
    failureThreshold: it will tell if the check is fail 3 times conitinusly then it will mark as failure.


one of the adavntage with k8s:
------------------------------

it will automatically add some tolerances to Pods:

1. node-notready
2. node-unreachable

* these taints will be applyed for the node and these tollerance will be applyed fro a pod.

* if the node is not ready or unreachable then pods will evict from the node safely and move to another node.



   
