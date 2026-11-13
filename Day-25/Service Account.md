Service Account:
==================

* A service account is a type of non-human account.
* They mainly used by Applications, Bots.
* Service account is also a user.
* we have to can create role/ clutser role we have to bind it to service account for access.

  Eg: Take jenkins server
  
  * This jenkins will be executed from the service account.



* if you create a service account it will be create without any secret / token.
* for the service account we have to create a token with the help of secrets.
* we have to use a anottation to attach the token to the service account: kubernets.io/service-account.name: serviceaccount_name
* we have to metion a secret type: kubernets.io/service-account-token.

* IF you create a pod with a service account the SA token will be mount on the pod/container in a specific path.
* Path: var/run/secrets/kubernets.io/serviceaccount
* in this above path we can see SA token, CA.crt and namespace

For Task refer the repo and do imagePullSecret for service account. for more infor check kubenets docs.
