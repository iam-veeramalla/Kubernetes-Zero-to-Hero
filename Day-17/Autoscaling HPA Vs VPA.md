 Autoscaling HPA Vs VPA
 =======================

In Autoscaling we have two trypes
-----------------------------------
   1. HPA
   2. VPA

First we will see about HPA:
-----------------------------
  1. HPA means- Horizontal Pod Autoscalling.
  2. In this case Pods will scale in horizonal way --------->
  3. means if the load or traffic incresses automatically pods will be add.
  4. based on cpu, memory, traffic.
  5. HPA Scaling means it will scale up the pods with the same resources and limits.
  6. there is no down time hear.
  7. in HPA we called pods will scale in and scale out
  8. it will comes with in kubernets.
  9. HPA keeps monitoring the mentioned resources by default time is 15 sec.

* In HPA we are scaling  for infra then we can called it as Cluster Autoscaler.
* Cluster Autocaling comes with cloud providers.


Second VPA:
------------
  1. VPA means - Vertical Pod Autoscaling.
  2. In this case Pods will scale vertically
  3. means if the load or traffic increasses the pod resources will be increassed.
  4. not like a HPA.
  5. in this case pod will restrt so we can see down time.
  6. In VPA we called pods will scale up and scale down.
  7. we have to install in k8s.it will not come with in k8s.

* In VPA we are scaling for infra then we can called it as Node AutoProvisioning.


Third Based on Events:
-----------------------
  1. it will a third party tool (KEDA).
  2. it will scle based on the events.
 
Note: 
-----
To implement this HPA we need mertics server. this metric server will expose the mentric of our node to HPA.

How we can manage multiple resources in a single yaml file.
------------------------------------------------------------

* In between the resources we have to give a line with (---) three hifens
  Eg:
      apiVersion: v1
      Kind: Pod
      -----
      --
      ports:
      - containerPod: 80

      --- (theree hifens) to devide the resources in a single yaml

     apiversion: apps/v1
     kind: deployment

  Refer the deployment file in Piyush repo


  Now we can see how we can autoscale the deloyment Pods in imparative and declarative way:
  -----------------------------------------------------------------------------------------

  imparative way:
  -------

  cmd: kubectl autosale deplyment deplyment_name --cpu-persentage=50 --min=1 max=10

  

  
  
