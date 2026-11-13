Node Affinity
==============

* Taints and tolerance have some limitations like we cannot add multiple conditions, expressions.
* it will not gurentee the pod will schedule on the tainted node. may be it will schedule on any of the untaineted node as well.
* it will give node to ablility to accept someset of pods and restrict some set of pods.

To overcome this few drawback we have NodeAffinity:
----------------------------------------------------

* By using this nodeAffinity we can schedule the pods on the mathing nodes with the help of labels we can give more contions using the operaters.
* Another advantage of this NodeAffinity we can use "required" and "preffered" blocks to use.
* With "required block" we can schedule the pod on the matched node only. after some time the node label was changed it will not imapct the pod which is running on the
  node. but the new pods will not schedule because label was not matching.
* With "Preffered block" we can schedule the pods on the matched labeled node and we can able to schedule the pod on the node which is not haveing the matching
  label. after some time if you add another lable to node also it will not imapct the pod which is running on the node.


  * Always use tain and tolarence with affinity togeather to make sure the pods will schuled on the specified tainerd node and we can prevent to schedule on anyother nodes.
    
  

