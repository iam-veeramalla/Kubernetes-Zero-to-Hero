 Role Based Access Control Kubernetes:
 ---------------------------------------

 * to provide authorization to user or a group in k8s cluster we will mainly use RBAC.
 * to implement this RBAC we will use 4 major components
   1. Role
   2. Cluster Role
   3. Role Binding
   4. Clutser Role Binding
  
  1. Role:
     -----
     * Role is a set of policies
     * in role we have few major thing we have to mention:
       1. resource Group: coregroups([" "]), apps/v1, batch.
       2. verbs: means what access we are giveing to the resources get, list, read, write, update.
       3. namespace if this role is spefic to ns.
          
 2. Role Binding:
    -------------
    * Role Binding is used to bind the role to Users, Groups, service accounts.
    * By using Role ref we can bind the role to above things.
   


CMD'S to list the Roles:
------------------------


To list roles in ns:
--------------------

kubectl get role -n name_space.

To list the roles in default ns:
-----------------------------

kubectl get roles

To list the roles in all ns:
-----------------------------

kubectl get roles -A

To count the roles in all the ns:
----------------------------------

kubectl get roles -A --no-headers | wc -l



Now we can see how we can login as other user:
-----------------------------------------------

Step 1: First have to add entry of the user on the kube-config

  CMD: kubectl config set-crendetials user_name \
       --clinet-key=path/.key \
       --client-certificate=path/.crt

Step: 2 Now set the context:

  CMD: kubectl config set-context context_name --cluster=cluster_name --user=user_name
  
Step 3: swith the context:

  CMD: kubectl config use-context context_name

To know which user i am have login:
-------------------------------------

CMD: kubectl auth whoami


* After set the context and use the context if the use is not able to authenticate the kube-Api we have to check the user certificate is valid
  or expired.
* if it is expired we have to renew it with opensll commands.






     
