Network Policies:
===================

* Every k8s cluster have there default network CNI plugin for the pods communication with in all the nodes.
* Means these CNI pods will be created as a deaman set in all the cluster nodes.
* So all the pods will communicate with eachother by default.
* but in production env this default communication is not preferable.
* so we can restrict these communication with the help of Network policies.

* we have some CNI plugins
  1. calico
  2. weave-net
  3. cilium
  4. flannel ---it will not support neworkpolices
  5. kindnet(come default with kind and minikube)---it will not support networkpolicies
 
Note: if your using a minikube or kind cluster we have to desable a default CNI then we have to install CNI from docs.

* in this case will take 3 tire application so it have frontend, backend and db.
* we have create 3 pods and 3 services for application.
* then we will implement network policies to restrict the communication b/w the pods.


* In the policy we have applied to db pods. so db pods will only allow the ingress trafic from the backedpods.
* fornt pods will not able to access the db pods.
* in production first we will create a denay so every pods will not comminicate eachother then we will set a networkpolices to enable communication
