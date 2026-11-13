Taints and Tolerations
======================

Taints and Tolerations : 
------------------------

* If you taint any of the node with a specfic taint label. this node will only able to schedule the pods which a toleration matched to the taint lable. remining     all the pods will not be schedule in this tainted node.
* in my cause i want to run AI workloads so for that i have a specific node which is having that capabilities. so in this node we want only those AI workloads need to run. so for that first we have to taint the node with a tainted label "Eg: gpu=true". now normal workloads try to schedule in this tainted node but this node will not allow them because the node is tainted with a specific label. Now we have a acutal AI workloads with the tolarence label "gpu=true" so node will tolarate those AI workloads to schedule on the node.


Effects:
---------

* we have 3 effects:
  1. NoSchedule (new pods): for cluster upgrades we will use this option
  2. preferNoschedule (no guarentree): by using the we are teeling the scheduler in the worest situation only schedule a Pod in this Node) this will use for            performence issue Nodes.
  3. noExecute (existing/ newpods): if you use this all the pods in the node will stop instently

To taint the node:
-------------------

cmd: 
     
      kubectl taint node node_name key=value:effect

* if you want to add the tolerance with the efffect in yaml. we have to add **tolerations** block in the yaml file.

To remove the Taint on the node:
----------------------------------

cmd: 

     kubectl taint node node_name key=value:effect-

we have to add ** - ** at the end of the command

Node Selector:
--------------

* By using this node selector we are allowing the pods to schedule on the matched labeled node.

* we have to use **"nodeSelector"** block in the yaml file.


To Label the node:
------------------

cmd: 

      kubectl label node node_name key=value



Simple Underastanding:
----------------------

Taint & Tolerance: by using this node is able to decide the pors are able to schedule in the node.
NodeSelector: by using this pod will decide to schedule on which node.
