 what is the diff b/w Container, Pod and Deployment ?
 ---------------------------------------------------

 container:
 ---------

  To running a container in docker we will use command in that command we will define 
  all the specifications.

  Pod:
  ----
  it is an Enterprice model hear we define all the specification of a container in yaml file.
  in pod will create single or multiple containers.
  Eg: sidecar containers called service mesh.

  Deployment:
  -----------

  By deployment we will get auto Healing and scaling capabilities.
  but pod don't have those capabilities.
  hear we are finally creating a Pob but not directly. for that we are using Deployment.

  If you create a deployment:
  --------------------------
  Deployement ----> Replica set ------> Pods
                      |
                      |
                    Controller

Replica set will create the Pods that we mentioned in the deployment.yaml file.
Replica set will make sure there is always defined replicas will run on the cluster.
Replica set controller will maintain the actual state same as desired state.

 

Replication controller, Replicaset, Deployment
==================================================

1. Replication Controller:
   ---------------------

   * if one app pod is running in the cluster so end user accessing that pod suddently pod creashed and access broken.
   * to overcome this issue k8s intraduces Replication controller.
   * this replication controller is one of the controller in k8s. it will make sure thet always maintain desired state of the pod count mentioned in the Replicationcontroller.
   * if i mentioned 3 replicas it will maintain 3 pods in our cluster.
   * in this case we are re-directing traffic to replication controller.
   * this replication controller is responsible to split the traffic based on the pods health acts like a load balancer.
   * after some time i increassed a replica count to 3 to 4 so it will schedule a new pod but the node didn't have enough space so cluster auto scaling will spinup a new node and this replication controller will create a new pod on the new nodea and distribute traffic to 4 pod as well.

   Replication controller rc.yaml file:
   ----------------------------------
        apiversion: v1
        kind: Replication Controller
        metadata:
          name: first-rc
          labels:
            app: emp
        spec:
          tempalte:
            metadata:
              name: emp-pod
              lables:
                app: emp
            spec:
              containers:
                - name: emp-cont
                  image: nginx
          replicas: 3

cmd: kubectl create -f rc.yaml

replication controller is a legacy version


2. Replicaset
   -----------
   * it is a modern way of replication
   * the major diff b/w the replication controller and replicaset:
   * replication controller will takecare of the pods created by a rc.
   * replica set will takecare of the existing pods which we are created before with the same lable along with the replica set pods. we can do this with the help of selectors and matchLables.

 Replicaset rs.yaml file:
 ----------------------------------
     apiversion: apps/v1
     kind: Replicaset
     metadata:
       name: first-rs
       labels:
         app: emp
     spec:
       tempalte:
         metadata:
           name: emp-pod
           lables:
             app: emp
         spec:
           containers:
             - name: emp-cont
               image: nginx
       replicas: 3
       selector:
         matchLables:
           app: emp

cmd: kubectl create -f rs.yaml

* we can make the changes of the objects in yaml file or a live objects.
  
* if you want to do in live object:
  ---------------------------------
   kubectl edit rs first-rs
* by modifiny the objects in live we no need to apply to it will automatically reflect
  
* if you want to sclae up the pods in rs with imparative way:
  -------------------------------------------------------------
   kubectl scale --replicas=10 rs first-rs

Note: it you want k8s help: kubectl scale --help  ---> it will show all the cmds.


3. Deployment
   ----------

   * Deployment is a advanced version. if you create a deployment the deplyment will create --> replicaset-->replicaset manges the--> pods.
   * by using deployment we have more advantages.
   * if the pods are running with 1.1 version. now we have version 1.2 so we have to update the pods with 1.2.
   * for this case if you use replicaset it will apply changes at all the pods at a same time. so users will face downtime.
   * for this case if you use deployment it will create a new version pods in rolling-update model. it will update pod by pod.


Deployment dp.yaml file:
----------------------------------
     apiversion: apps/v1
     kind: Deployment
     metadata:
       name: first-dp
       labels:
         app: emp
     spec:
       tempalte:
         metadata:
           name: emp-pod
           lables:
             app: emp
         spec:
           containers:
             - name: emp-cont
               image: nginx
       replicas: 3
       selector:
         matchLables:
           app: emp

cmd: kubectl create -f dp.yaml

* if you want to modify objects in deployment:
  -------------------------------------------------
* by editing the yaml file directly
* by editing live object via cmd: eg: we need to update a image: kubectl set image deployment first-dp old_image_name=new_image_name

* if you want to see rollout history:
  -----------------------------------
  
       kubectl rollout history deployment deployment_name

* if you want to rollback changes:
  ---------------------------------
  
      kubeclt rollout undo deployment deployment_name

* To see more detail about a particular revision:

      kubectl rollout history deployment nginx --revision=2

* How do we check rollout status:

      kubectl rollout status deployment nginx


       | Command                                                 | Purpose                          |
       | ------------------------------------------------------- | -------------------------------- |
       | `kubectl rollout status deployment nginx`               | Check rollout progress           |
       | `kubectl rollout history deployment nginx`              | View revision history            |
       | `kubectl rollout undo deployment nginx`                 | Roll back                        |
       | `kubectl rollout undo deployment nginx --to-revision=2` | Roll back to a specific revision |
       | `kubectl rollout pause deployment nginx`                | Pause rollout                    |
       | `kubectl rollout resume deployment nginx`               | Resume rollout                   |



Note: 

     if you want to create a yaml file from the imparative command
     Kubernetes keeps Deployment revision information so you can move back to a previous version when needed.
     Deployment
     │
     ├── RS-v1 → nginx:1.25 → 0 Pods
     ├── RS-v2 → nginx:1.26 → 0 Pods
     └── RS-v3 → nginx:1.27 → 3 Pods
     

     



kubectl create deployment deploymet_name --image=image_name --dry-run=client -o yaml > dp.yaml


     
   

   
   




 
