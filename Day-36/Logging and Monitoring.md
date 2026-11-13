 Logging and Monitoring
 ========================

* in kubernets we monitor the resources help with the help of "*metrics-servers*" and "*metrics-api*".

Q)  How this Metrix server will help the cluster ?
A)  the Metrix server will help k8s to manage HPA pod autoscaling based up on the CPU and MEMORY. The second one when we run the kubectl top command to check the 
    k8s resources CPU and MEMORY utilization.

* Then what happen when we run the Kubectl Top command and when we enable the HPA pod autoscalling
* internally the process is in worker plane we have a componend called CAdviser this CAdvise will collect the metrix from the container_run_time like containerD and
  it will forword those metrics to kubectl this kubeclt in worker node forword these node level metrics to  "*metrics-servers*" these metrics will be
  exposed to kube-api server with the help "*metrics-api*" interface.

installtion:
------------

step:1 
-------
go to this below github repo:

       https://github.com/kubernetes-sigs/metrics-server

step:2
-------
Download the yamlfile but at the time of applying the yaml file we have to add -kubelet-insecure-tls this args in the commads portion:

     kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


disable certificate validation by passing --kubelet-insecure-tls to Metrics Server.



step:3 
------

now if you run the kubectl top command we will see the resource utilization logs of the resources.

       kubectl top pod


Now we can disscus about "*circtl*"
------------------------------------

* This circtl is the ctl tool to interact with the containerD.
* we will use this circtl at the time of troubleshooting the issues when the kube-api server is down.
* we can use this circtl same like a docker commands.

Refer the below link for more circlt use cases and commends:


      https://kubernetes.io/docs/tasks/debug/debug-cluster/crictl/



