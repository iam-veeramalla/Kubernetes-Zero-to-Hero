 ARCHITECTURE OF ARGO CD
 ========================

 1. Repo server: repo server main job is connect to git and get the state of the git. it is one of the micro service help to connect with the git.
 2. Application controller: Application controller main job is connect with k8s cluster and get the state of the k8s cluster.

                  it will do another job once repo server get the state from the git. this app controller will compare both the state.
                  if there is any change the app controller will fetch the changes and implement in the cluster.

3. Api server: used to intract with the Argo-CD via UI or CLI. it will handel the authentication as well. we can have a single sign out option and can integrate with
               existing OIDC providers or user login DB'S. There Dex comes into a picture.
  
4. Dex: By using this we can integrate with OIDC fro authentication.

5. Redies: is used for catching the information.


 Real world Q&A
 ---------------

 1. i have a pod.yaml in git. i deployed with the help of ArgoCD. as a treditional behavior of k8s admin controllers will add some taintes, tollerences, resources limets.
    is this chages cases any issues ?

2. before using the argocd we deployed two applications in the cluster is those app will be delete after adding the argo-cd ?
    
