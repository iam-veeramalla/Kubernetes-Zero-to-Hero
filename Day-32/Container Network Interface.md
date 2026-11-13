Container Network Interface CNI :
=================================

First we will Talk About Docker Networking:
-------------------------------------------

* In docker if you run Docker run what will happen ?
* The container will be created.

* First Thing Dockcker have
  -----------------------------
  1. Docker daemon --> it will not start the container.
  2. Containerd --> it will maintain life cycle of the container
  3. Docker Shim --> it will ensure the communication B/W the containers and containerd
  4. runc --> this runc a shotling proccess the responsible for the container start.


* Second comes to Kubernets Networking
  -----------------------------------
  1. hear we can make a api request like cretae a pod.
  2. secheduler will check the sutable pod.
  3. kubeapi server will send that request to kubelet --> Hear kubelet is like a docker daemon
  4. kubelet will make a CNI call.
  5. the CNI is communicate with container runtime ---> Shim---> runc will start the container. same like docker 
