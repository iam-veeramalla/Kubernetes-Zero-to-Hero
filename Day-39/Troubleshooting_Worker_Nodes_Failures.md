Troubleshooting Worker Nodes Failures
======================================

* if you're worker nodes are **Notready** condition.


Case 1 in this case we have to check few cases
-----------------------------------------------
  1. Network plugin failure.

* we have to check the network pods are running or not.
* Command:

           kubectl get pods -A or kubectl get ns
  
* There we have to check the calico pods are running or not for calico we have to check tigero-opertater is running ot not in tigero ns.
* Then we have to check calico-system and calico-apiserver ns.
* The default path of the cni plugins:

               /etc/cni
* in the above case all the calico pods will be running file then we have to elemenate the case.

Case : 2 Next case we have to ssh to the worker node with ssh <username> <worker_node_name>
-------------------------------------------------------------------------------------

* in this case we have to check the kubelet is working or not.

       service kubelet status

* if it is in inactive state we have to restart it.


       service kubelet start

* the we can check the node status from the controle plane. you will see the node status in ready state.

Case: 2  Next we can see another case for this node_not-ready state:
-----------------------------------------------------------

* when i ssh to worker node i have check the kubelet is active or not. in this case i have seen the status is **activating** and these is no logs.

* so first i have checked the kubectl logs.
* to check the services logs i have to use journalctl

        journalctl -u <service_name>
* in the logs if you go to the last line with the help of shif+g
* in the last line logs we could see some I and E type of error codes.
* I = informative
* E- error.
* we have to check the error.

Error: 

         unable to load client CA file 

* so i have to check the kubelet config file.

         cd /var/lib/kubelet
* in the kubelet folder we have to check the config.yaml file.

* in the file we have to check the clientCA row and we have to crocess check the ClientCA file name is correct or not. to crocess check that.

* we have to go:

        cd /etc/kubernets/pki

* if the Clinet CA name is wrong we have to correct the client CA name in kubelet/config.yaml file in the /var/lib/kubelet folder and we have to
  restart the kubelet service:

               service kubelet restart


  

  


