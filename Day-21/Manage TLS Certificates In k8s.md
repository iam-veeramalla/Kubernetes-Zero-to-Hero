Manage TLS Certificates In a Kubernetes Cluster
================================================

* In k8s we have to make sure the communication b/w the client to API -- API to Contarol plane components -- API to woler plane components.
  control plane components to API server must be always secure.
* To implement security we use HTTPS protocal for communication.
* So every componet in k8s have there own certificate and private key signed by CA.
* client have there own certificate and private key signed by CA.
* Evnen CA have there own root certificate and PrivateKey.


scenario: 1
-------------

* If user is requested the kubeapi Server.
* User is (Client) ---->>kubeAPI is (Server)
* if kubeapi is communicating with etcd.
* kubeapi is (Client) ---> ETCD is (server)
* In this way based up on the flow the clint and server positions will change in k8s.


Now we can see how a new user in organization will access the cluster:
----------------------------------------------------------------------

New User steps:
---------------
 Step:1 - New user need to creare a privati.key by using Openssl.  -- output: private.key was created

 Step:2 - By using this key we have to make a new CSR (Certificate signing request) -- output: .crt file was created this is CS request.


Cluster Admin steps:
---------------------

step:1 - we have to create a yaml file with kind "CertificatesignatureRequest" and in the spec: request: we have to give the CSR- key.

Step:2 before  give the CSR- key we have to convert that into base64 and in the file we have to give in a single file.

        cmd: cat file_name.crt | base64 | tr -d "\"

* As of now we just created a object CSR in k8s. if you see by using kubectl get csr--> the request object in pending state.
* Now this CSR need to approved from private CA.
* In my local k8s cluster i will use server certificate and private key as CA to approve CSR.

* To approve the cert we have a certion role permissions.

  cmd to approve the request:
  ---------------------------

  cmd: certificate approve csr_name

* once certificate was genetared we have to give the certificate to user in decrption formate then user will add that in kubeconfig.



 
 
