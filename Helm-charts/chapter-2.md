Healm structure ?
-----------------
* in helm chat mainly we can have 3 components
  1. helm cli
  2. helm repo
  3. helm charts

1. Helm Cli
   --------
   Helm Cli will check the kube config file and it will know to which cluster it needs to communicate it.

   Main 3 helm basic commands:
   1. helm install
   2. helm delete <chart_name>
   3. helm list -a

Now we can see how to cretae a Helm chat:
-----------------------------------------

To create a Helm chat: 

       helm create <chart_name>

* if you run the above cmd you can see helm chat was created with required yaml files like values.yaml, chart.yaml files and chars and templates folders.


To see the strcture of Helm Chat:

             tree <chart_name>

* For service configration we have to go to "values.yaml"  ---> find the service block and update there you'r service_type and port.

* To install the helm chat we have to use *helm install*:

          in helm install CMD we have two orguments:
          1. Release_name ----> This Helm chat name which we are deploying to cluster. we can give what ever we need hear.
          2. chat you want to install. -- > the actual helm chat.


         CMD: helm install <release_name> <actual chat name>


* if you do helm list -a you will see the list of helm chats were installed in the cluster with the release_name

* To uninstall helm chat:

          helm unimstall <release_name>
  





 
    
