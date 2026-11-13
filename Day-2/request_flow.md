1. user requested for Pod creation. the request will go to Kube-Api-server.kube-api-server will authenticate, modify and validate 
    the rquest of the user. if the request will all these check points.

2. then the request will go ETCD and the ETCD just make the entry of the new pod request and respond back to kube-api- then kube-api sends the request to scheduler. 
3. scheduler will check with the node kubelet
4. kubelet check the node resources and report to schedler i have enough resources and proper policies to run this pod.
5. then the scheduler pass these information to kube-Api-server
6. kube-api-server will communicate with the kubelet in the worker node for pod creation.
7. kubelet will create the pods in the worker node
8. and inform back to kube-api-server pod creation was completed.
9. so kube- api- server will only component will communicate with etcd. kube-api-server will store this pod information.




