Static Pods, Manual Scheduling, Labels, and Selectors:
======================================================

Static Pods:
------------

* In any of our k8s cluster's a control plane component called scheduler will takecare of scheduling a pods in our worker nodes based up on the scheduler algoritham.

 1) But How the control plane components which are runnig like a pods in owr cluser ?
 A) These control plane componets will schedule with the help of static pods.

 2) what is this static Pods ?
 A) Static Pods are the mainly a control plane componensts which are scheduled by a kebelet not a scheduler.

 3) How this kubelet schedule these Pods ?
 A) Kubelet will check the specifig directry in that if it find any .yaml file with configration the kubelet will spinup the Pod.

* Kubelet always monitore the below mentioned directry.

* In controlplane /etc/kubernets/manifest/ ---> Directry all the control plane static pods yamal file will be available.

Manual Scheduling:
------------------

* Take a scenario like you have 2 pod yaml files 1 yaml file have a field called "node selector" and 2 pod yaml don't have a filed called "node selector".
* In this case scheduler will take care about the pod which don't have the filed called "node selector".
* If the Pod have a filed called "node selector" there is no help need from scheduler.it will automatically schedule in the mentioned node.

Note: once we scheduled the Pod we can't move the pod to specific Node. If you want to do that then we have to delete the existing pod and create a pod with node selector.



Labels and Selectors:
---------------------

* Lable is helpful in filtering the resources.
* Take an example of deployment:
* In the deployment.yaml file we specify thelables in 3 portions.
  1. in deplyment metadata portion which is relavent to deployment.
  2. in spec the label is related to pod.
  3. in selectore portion. this selector will matching the pod lable with the deplyment lable. so if the label match it will maintain the pod as part of the deplyment.





