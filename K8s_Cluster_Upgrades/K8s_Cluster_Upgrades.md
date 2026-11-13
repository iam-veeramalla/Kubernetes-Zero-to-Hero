Kubernets Cluster Upgrades
=============================

In this section we will see how to upgrade the Enterprise level Kubernets Cluster with 0 downtime.

Upgradation is one of the Task of Devops Engineer in every 3 months.

Every 3 months K8s comes with new version.

K8s only support to latest 3 versions. Some time cloud provider will support old versions as well.

Before Doing cluster Upgrades we have some prerequistes:

Prerequistes :
--------------
if we are in Production Environment.
we have to inform or communicate to other team we are planning clutser upgrades so it will take 
1 - 2 hrs of time so there is no impact on existing services 
but 1 to 2 hrs we are stopping of any new deployments.

1. Cordon Nodes: means makeing our nodes unschedulable during the process of clutser upgrade.
2. Release Notes: we have to verify the Release Notes of the K8s latest version
   Eg: in old version our ingress using API group extenstion/bets1 but in new version
       it was changed to networking.io/v1 if you didn't observer this change in relase notes
       it will makes some issue with ingress.

3. Downgrade: we can not downgrade our k8s cluster version specally for "EKS".
   Eg:we have upgrade our K8S cluster from 1.30 ---> 1.31 but in the 1.31 we found some is
      not working if you want to get bact to old version again we have to do Fresh installation
      means rollback is not possible.it will impact organization.
   
4. Lower environmet: to overcome the above issue have to do upgrade in lower environments first and
                      we have to check the behavior at least one to 3 weeks.

Technical Prerequisites:
------------------------
1. Control Plane and Nodes:Make sure our control plane and worker nodes should be running with same version.
   Eg: Control Plane is running on version 1.30 and Nodes are running with vesion 1.29.
       in this case before upgrading the Control Plane from 1.30 to 1.31. first we have to upgrade
       our worker Nodes from 1.29 to 1.30

2. Cluster Auto Scaler: if you are useing Cluster Auto Scaler we have to make sure that Cluster Auto Scaler
   version must match with vesion of Control plane.

3. we have to make sure at lease 5 available IP address in the Subnet that we have provided during
   the cluster creation.

4. kubelet version is alos match with control plane
   
    

Process of Upgrading EKS Cluster:
----------------------------------
Step:1 Managed Control plane upgrade  --->(time: 30min)
-------------------------------------
Note: one thing to remember our EKS is managed cluster means Aws will take care about Control plain components and high availability, desaster recovery, security and scaling of API server.
      "but it will Not manage control plane upgradations"

1. For Managed Control Plane upgrades we can do it via (UI, eksctl, aws cli).
2. once we click in update on UI or fire any eksctl commands then AWS will takecare of upgrading the CP.

Step:2 Node Group Upgrade or Nodes or Fargate ---> (time based on node count)
-----------------------------------------------------------------------------
1. Managed Node Groups: it was managed by AWS. by using eksctl command we can create Node Groups launch template. 

with Mnaged Node Groups the upgration was simple just we have to select "Rollout" option. the the managed k8s service will go to Node group and upgrade node by node. if you have more than 100+ node this approch us better.

the upgrade involves in lauch templates or AMI. 
if we are using our own launch templates it's our responsible to upgrade launch template.
if we are using Custome AMI it's our responsible to build custome AMI.
if we using the AMI provided by AWS in this case we just click in Upgrade.--> update stretegy--> "  Rolling Update"


2. Nodes: Managed by own for this we have create one custome launch template or they will create ec2 instances and attach them.


This is very tricey process:
1. cordon the node: means make the node unshadulable.
2. upgrade that node




3. Hybrid: combination of above two




step3: Addond Upgrade:---> (time based on addons count)
-------------------------------------------------------
like Kube-proxy, VPC CNI 


One Cooman QA:
--------------

Q) i have installed HELM, prometheus, ArgoCD do i need to stop all of them at the time of upgrade ?

A) to make sure we have to verify they are working or not after upgrade. so every prod upgrade we 
  have to updrade lower environment and make sure thay are working fine are not.


  

By the end of the Day our QA team have functional testing script on upgraded version.

no need to go with individual componenets.










