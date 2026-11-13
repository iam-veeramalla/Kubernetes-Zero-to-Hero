In this section we can discuss:
1) Aws EKS Cluster Authentication & Autherizatio 
2) How to connected multiple clusters from User Mechine.

   

*  Aws EKS Cluster Authentication & Autherizatio :
--------------------------------------------------
The below steps will we need to perform in the users mechine who need access to AWS eks cluster:
who create the EKS cluster by default that iam user will be add in Cluster "IAM ACCESS ENTRIES".
--------------------------------------------------------------------------------------------------
* First we have install AWS CLI and Kubectl in Users Mechine.
* After installing the Kubectl if we Run any command related to Kubectl, first it will search for a      file called "config" where this file will availble in our mechine in "root folder/.kube/config"-->  t
  this the actual path where the config file was available.
* what this config file contains ?
* In config file we have the info to which cluster we have to connect. if this file was not there in  our mechine then our kubectl request will not reach to our cluster.
* How we can create that Config file our mechine ?
* To create that config file we have to use a command:
  aws eks update-kubeconfig --region clutser_region --name cluster_name
* then it will throw an error message:
  YOU CAN CONFIGURE CREDENTIALS BY RUNNING "aws configure"
* So first we have to create a user in AWS IAM.
* At the time creating user in AWS IAM console what are permissions we are providing to user is only applyed at AWS console level. not applied to AWS EKS cluster.
  Means if you provide EKS access at the console leavel he will able to see the cluster but he will not able to access it.
* Once we have creted the user we have to create "Access key" and "secret access key".
* now again come to user mechine and configure these key's by using the command:
   Commad: aws configure
* once we have configure the Aws keys in our AWS cli now create the config file with the command:
    aws eks update-kubeconfig --region clutser_region --name cluster_name
  
* so the config file was cretaed with Clutser details and pointing to the cluster.

* in that config file we have our clutser data called:
   
  clutser certificate authority
  clutser ARN

* now we can run the kubectl command to see the output. But still we are unable to see cluster resouces. Why ?
* Because the IAM user have only access to AWS console so user can see the resources in the AWS console. But if he want to acces the Aws EKS cluster he can't.
* For Aws EKS clutser Access we have to add that IAM user in Cluster "IAM access entries"
*  If we want to check Which user is configured in the mechine:
   aws sts get-caller-identity
   Result: we see UserID, Account, Arn of the user.
*  we can give IAM access entries in two ways:
   1. EKS API
   2. EKS API and ConfigMap
*  How to Provide cluster access to the IAM user.
   1. click in create access entry
   2. provide IAM user Arn
   3.Type: Standerd
   4. Policy name: select cluster policy
   5. Acees scope: cluster or namesapce
   6. create
*  Bases on the policy added in the access entry user is able to Authorisation the cluster.

Now we can see How to connects Multiple EKS Clusters:
======================================================
 
1. how we can see how many clusters(contexts) configured or connected to our mechine ?
A. Command: kubectl config get-contexts
   Result: we can clusters details connected to our mechine.


ok we know some of the clusters connected to our mechine.now i want to which cluster my mechine getting the detsils.

2. How we can find from which cluster my mechine is fetching all the details ?
   Commad: kubectl config current-context
   Result: it will show to which EKS cluster our mechine is pointing to fetch the details.

3. if you want to see mechine pointing cluster info:
     Command: cat config

if you want to mange multiple clusters from you'r mechine:
1. we have to add that cluster details to our config file.
   Command: aws eks update-kubeconfig --region cluster_region --name cluster_name
   
2. At the time of adding the cluster in config file we can set alias name for our cluster.
    Command: aws eks update-kubeconfig --region cluster_region --name cluster_name --alias our_own_name.


3. How to fetch details from another cluster which is configured in config file ?
    command: kubectl --context=cluster/alias_name get all

By using the above command we always tells kubectl to point to specific cluster to get the details.

4. if you want to swith to another cluster which is configured in the config file ?
    Command: Kubectl config use-context cluster/alias_name

so in kubectl commands we no need to provide cluster details.

 
 
 






   




