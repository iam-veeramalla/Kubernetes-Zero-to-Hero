        ✅What is Amazon Elastic Kubernetes Service?
        ✅How does Amazon EKS work?
        ✅Lab Session - Creating an AWS IAM User
        ✅Lab Session - Creating an EKS Cluster through the AWS Console
        ✅Lab Session - Adding Worker Nodes to EKS Cluster
        ✅Lab Session - Connecting to the EKS cluster
        ✅Lab Session - Deleting an Amazon EKS Cluster


✅What is Amazon Elastic Kubernetes Service?
---------------------------------------------

      * Amazon Elastic Kubernetes Service is a managed service AWS will take care of Master Nodes and master node components.
      * It will give high availability and scalable futures.
      * we will integrate the EKS cluster with various resources like IAM, Cloud watch and Load balancers.
      * it will give security futures like VPC isolation, IAM roles for service account and AWS KMS.
      * it will support both worker nodes and master nodes scaling.
      * it will support add on like AWS VPC CNI PLUGIN'S

✅Lab Session - Creating an AWS IAM User
-------------------------------------------

      * IAM user who have admin access will be create AWS eks.
      
      * in real world we create our eks or kubeadm clueter in private subnets.
      
      * we have to create one IAM role for a cluster with AWS_EKS_CLUSTER_POLICY

Clusrer-IAM role creation:
---------------------------

    Trusted entry: Aws service --> EKS
    Use case: EKS --> EKS_CLUSTER
    Policy:  AWS_EKS_CLUSTER_POLICY, AWS_EKS_NetworkPolicy, AWS_EKS_EBS_Strorage_Policy, AWS_EKS_Compute_Policy, AWS_EKS_LoadBalancer_Policy
    Role_Name: Role_Name


EKS Clutser Creation:
---------------------

      Click on create a cluster:
      
      Name: name of the cluster eg: Empower_Pord_cluster
      Cluster_Role: created in the above step.
      Kubernets Version: selectect last version not latest.
      Upgrade Policy: Extend and standerd --> select standerd and for cost opt update cluster every 3 months.
      EKS Auto Mode: need to learn
      Control Plane Scaling: need to learn
      Controle Plane configration: need to learn
      Delete protection: need to learn
      ARC Zonal shift: need to learn
      Cluter access: 
      Bootstrap cluster admin access: Allow cluster admin access
      cluster authentication mode: EKS api and configMap

Networking:
------------

      VPC: Select the vpc
      Subnets: select private subnets min 4 to 6 for high availability.
      Security Group: select default because once we create the eks cluster it will create one security group. in real time we use seperate sg.
      Cluster IP address family: IPV4
      Cluster end point access: Private--> to connect to this we have used client vpn to connect.

Configuring Observbility:
-------------------------

    Enable Promitheus or cloudwatch
    enable network monitoring
    Control plane logging: Enble all logs of master plane.



Add-on's:
---------

    1. CoreDNS
    2. Kube-Proxy
    3. Amazon VPC-CNI --> need iam role with policy **AWS_EKS_CSI_Policy**
    4. CSI snapshot drivers
    5. EBS CSI drivers -----> need iam role with policy **AWS_EKS_EBS_Strorage_Policy**

Configure-selected add-ons settings:
---------------------------------------

    hear we can select the verions for the above add on's
    
    
    * Once every thing is selected we will review all the settings in the last window and click on create cluster it will take 10 to 15 min of time to create the cluster.

Once we created the Cluster:
----------------------------

in over view tab:
------------------

    1. we will see API Server Endpoint
    2. OpenID Connect Provider URL
    3. When it was created
    4. Certificate authority
    5. Cluster IAM role ARN
    6. Cluster ARN
    7. Platform Version--> it will represent the path version in the major version like v 1.30.
    8. Kubernets Version settings: ---> Upgrade policy: standers/ Extended
    9. Health Issues
    10. secret encryption

in Resources Tab
-----------------

    1. we can see our workloads like deployments, pods, configmaps, secrets, daemonsets, statefull sets, HPA, Jobs.
    2. we can see default namesapces like default, kube-node-lease, kube-system, kube-public.
    3. Nodes: how many nodes we have

in compute Tab
--------------

    * Worker load realeted nodes.
      
    1. hear we can see the node groups
    2. fargate server less compute details

in networking Tab:
------------------

    1. VPC details
    2. Subnets detsils
    3. security group details: we can see EKS created SG aling with our sg while we provided at the time of creation. in eks generated sg ingress and egress allows all traffic

in Add-ons Tab:
---------------

    * we will see all add-ons we have selected at the time of creation. if you didn't add any node to eks these add-ons will in creation state.


in Access tab:
--------------

    * Access configrations: EKS API and configMaps
    * Pod idenetity access:
    * OIDC identity provider:


Observbility:
-------------

    * if we enable promitheus we can see mentric in this tab

Notes:


       1. we can upgreate clutser from 1.28 to 1.30. because at a time we can upgrade to one version we can not skip the versions.
       2. we can degreade to previos version as well.
       3. before applying the upgrate to dev, stage and prod we have to go with poc.
       4. if there is any issue with the master node we will check with aws by rasing a ticket.


Now we see how to add worker nodes to EKS Cluster:
--------------------------------------------------


IAM role creation before creating the worker node:
---------------------------------------------------
    Trusted entry: Aws service --> Ec2
    Use case: EKS --> Ec2
    Policy:  
    Required policies:
    
    1. Amazon Eks WorkerNode Policy
    2. Amazon ec2 container rigistery Read only policy
    3. Amazon EKS_CNI_Policy
    4. Amazon elastic_container_rigistery_Public Read only policy

    Role_Name: Role_Name

Create Node_Group:
------------------

    Go to EKS --> Go to Compute tab --> click on create node group 

Step: 1 Configure Node Group:
------------------------------
    Name: name of the node group
    Node_IAM_Role: select the above node group.
    Option one: real world we slect the lauch templete which have the custome ami with amozon linix EKS.


Note:   

        don't select normal AMI'S LIKE amazon 2 or amazon linux 2023
        only select Amazon eks linux 
        for eks ami's we can go *amazon eks ami relese*
        there we can find the specifc eks version related ami's 
        
Launch template: select the existing launch template

Option two:
set compute and scaling configrations:
In this section we can select:
AMI type:
capacity type:
instance_type:
Disk size



Kubenets taint's: 
Kubenrts labesl: 
Tgs:

set compute and scaling configrations if we are going with option two:
------------------------------------------------------------------

AMI type: Go with Amazon linux 2 
Capacity type: on demand --> in real time
instance type: t2.large
Disk space: 100 min

Node group scaling configrations:
---------------------------------

Desired size: 
Max size:
Min size: 

Node Group upgrade configrations:
----------------------------------

Max unavailable:

Chosse *Number*: 1


Specify network:
----------------

Select the subnets for worker nodes. which we have selected at the time of eks cluster creation.

Note: Always configure remote access to worker nodes.

EC2_KEY PAIR: provide key pair.

Allow remote access from specific security group: select a specific security group

* Review and create a worker nodes. it will take 10 to 20 min to create the nodes.


In node group details tab:
---------------------------

Node Group ARN:
Autoscalling group name:
Node group IAM role ARN:



Now we can see how to connect to EKS cluster from cli:
------------------------------------------------------

* First we have install AWS CLI and Kubectl in Users Mechine.

* Configure AWS credentials

* install kubectl

* After installing the Kubectl if we Run any command related to Kubectl, first it will search for a file called "config" where this file will availble in our mechine in "root folder/.kube/config"--> t this the actual path where the config file was available.

* what this config file contains ?

In config file we have the info to which cluster we have to connect. if this file was not there in our mechine then our kubectl request will not reach to our cluster.

* How we can create that Config file our mechine ?

To create that config file we have to use a command: aws eks update-kubeconfig --region clutser_region --name cluster_name




