HOW TO INSTALL AND USE ARGO CD
===============================


1. go to official docs: https://argo-cd.readthedocs.io/en/stable/getting_started/
2. https://argo-cd.readthedocs.io/en/stable/getting_started/

        kubectl create namespace argocd
        kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

3. We know Argo-cd have an API server to communicate via CMD/UI.

4. once we installed the Argo-CD we will see all the Argo-CD related pods and Services in Argo-dc name-space.

5. we have to find the argo-cd api service and edit the argocd-svc to Node-port. because by default it will comes with CusterIP type.

6. after modifing to nodeport mode we have to port forwordig as per the cluster we are using.

7. if we are using a load balancer we don't need to do port forwording 

8. then we are able to access it with UI.

9. After accessing the UI we have a password. for that we have to get the secrets from ardo-cd namespace.

10. if you edit the argo-cd admin secret we will see the password in encripted formate.

11. to decript it use cmd: echo <passwoed> | base64 --decode -->except % rest all copy and past in password tab. username: admin

12. For practise use this git repo: https://github.com/argoproj/argocd-example-apps

Now Actuall work starts in Argo-CD:
------------------------------------

1. We have to create application hear. but we have to learn about application set generater.
2. bacis level we have give appname, syanc policy, source: git url, revison-brach, path,  cluster url and namespace.....click on create.
3. it will start deploying the application.
4. if you want to delete the application in argo-cd justgo with forground option.

 To Access the Aro-cd with CLI:
 -------------------------------

 1. we have to download CLI 
