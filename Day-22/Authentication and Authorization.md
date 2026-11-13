Authentication and Authorization
=================================

1. Authentication:
   ---------------

   * when we run the kubectl commands. this kubectl first refers the kube-config file for the user, clutser and certificate details.
   * this kube config file is the mail config file in uses mechine.
   * By default we will get this config file
   * in this config file we can add so many clutsers, users and context details along with client-certificate and client-key.
   * we can maintain more than one kube-config file so at that time we have to specify in the kubectl command to which kube-config it has to
      refer to execute this command.


   Multiplue cluster configration Ref:
   -----------------------------------
   apiVersion: v1
clusters:
- cluster:
    certificate-authority: fake-ca-file ----> this is cluter CA path we have to give hear
    server: https://1.2.3.4
  name: development
- cluster:
    insecure-skip-tls-verify: true
    server: https://5.6.7.8
  name: test
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
- context:
    cluster: development
    namespace: storage
    user: developer
  name: dev-storage
- context:
    cluster: test
    namespace: default
    user: experimenter
  name: exp-test
current-context: ""
kind: Config
preferences: {}
users:------------------------------------>      user
- name: developer
  user:
    client-certificate: fake-cert-file ------>   client cert
    client-key: fake-key-file  ----------->      client key
- name: experimenter
  user:
    # Documentation note (this comment is NOT part of the command output).
    # Storing passwords in Kubernetes client config is risky.
    # A better alternative would be to use a credential plugin
    # and store the credentials separately.
    # See https://kubernetes.io/docs/reference/access-authn-authz/authentication/#client-go-credential-plugins
    password: some-password
    username: exp


2. Authorization
   -------------

   1. ABAC : if we use this as Authorization we have to update this policy file in kubeapi server then we have to attch this to user and it's
             need kube-api server restart.
   2. RBAC : now a days we are using this a mainly in our kubernets clutsers fro autherization.
   3. Node : Node authorization we can use it for node to node access
             Eg: if kube-api want to intact to kubelet to do some task this node rbac will help us.

   4. Webhook: by using thirdpath we can do Authorization in clutser.

Kube-Api-Server:
----------------

* if you describe the kube-Api server file we can see some parameters like authorization mode.
* in Authorization we could seen Node and RBAC based on this sequence it will check the kubectl request.
* in  authorization mode if you specify allow all the authenticatores will allow do all the thing.
* if you specify the always denay all request will denay.
* so best practice is RBACK, Node, webhooks.

* In control-plane we have this kube-api configration. these config will refer by the kube-api server.
* in controle-palne we have all the componets key's and certificates.
* these keys and certificates will be availble in /etc/kubernets/pki-----> folder.

  


