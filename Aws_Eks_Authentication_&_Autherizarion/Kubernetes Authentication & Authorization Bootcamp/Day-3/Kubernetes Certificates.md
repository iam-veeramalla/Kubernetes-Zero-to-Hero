Kubernetes Certificates
========================

Before learning certificates, understand this question:

How does the Kubernetes API Server know that the person running kubectl is really Aravind and not someone pretending to be Aravind?

The answer is Certificates.

Step 1: Everyday Example
-------------------------
When you visit your bank:

You show your Aadhaar Card or Passport.
The bank verifies that the ID is genuine.
Only then are you allowed to access your account.

Kubernetes works similarly.

Instead of an Aadhaar card, it uses a Client Certificate.

      You
       │
       │ Show Identity Proof
       ▼
      Bank
       │
      Verifies
       │
      Access Granted

Kubernetes:

    kubectl
          │
    Client Certificate
          │
    API Server
          │
    Verifies Certificate
          │
    Authenticated

Step 2: What is a Certificate?
------------------------------
A certificate is a digital identity card.

It contains information like:

    Owner Name
    
    Public Key
    
    Issuer
    
    Expiry Date
    
    Signature

Example:

    Subject:
    CN=aravind
    
    Issuer:
    Kubernetes CA
    
    Valid From:
    2026
    
    Valid To:
    2027

Step 3: What is a CA (Certificate Authority)?
---------------------------------------------

Imagine your company issues employee ID cards.

Only IDs issued by the company are trusted.

   Company HR
         │
   Issues Employee ID
         │
   Employee

In Kubernetes:

     Certificate Authority (CA)
             │
     Signs Certificate
             │
     Trusted User

The CA is the root of trust.

Step 4: Why Do We Need a CA?
------------------------------
Suppose anyone could create their own certificate.
Fake User

Creates Certificate

"I am admin"

If Kubernetes accepted that certificate, anyone could become an administrator.

Instead:

API Server

Checks:

     Was this certificate signed by MY CA?

If:

YES

Authenticate.

If:

NO

Reject immediately.

Step 5: Important Certificates
-------------------------------

A Kubernetes cluster contains many certificates.

1. CA Certificate

         ca.crt

Purpose:

Signs every Kubernetes certificate.

Think of it as the Principal's signature on school certificates.

2. API Server Certificate


       apiserver.crt

Used by:

kube-apiserver

Purpose:

Proves to clients that this really is the Kubernetes API Server.

3. Client Certificate

Example:

      admin.crt

Used by:

     kubectl

Purpose:

Identifies the administrator (or another user) to the API Server.

4. Kubelet Certificate

Every worker node has its own certificate.

Example:

worker-node-1

This allows the kubelet to authenticate itself when talking to the API Server.

Step 6: Authentication Flow
------------------------------
Suppose you run:

kubectl get pods

The flow is:

            kubectl
                  │
            Reads kubeconfig
                  │
            Gets client certificate
                  │
            Connects to API Server
                  │
            API Server checks:
            "Is this certificate signed by my CA?"
                  │
            YES
                  │
            Authentication Successful
                  │
            RBAC Authorization
                  │
            Returns Pods

Notice the order:
            
            Authentication
            
            ↓
            
            Authorization

Kubernetes never checks RBAC until authentication succeeds.

Step 7: Where Are Certificates Stored?
---------------------------------------
If you created the cluster using kubeadm, you'll typically find them on the control plane node:

/etc/kubernetes/pki/

Example:

            ca.crt
            ca.key
            
            apiserver.crt
            apiserver.key
            
            apiserver-kubelet-client.crt
            
            front-proxy-ca.crt
            
            front-proxy-client.crt

etcd/


Step 8: How Does kubectl Know Which Certificate to Use?
--------------------------------------------------------
It reads the kubeconfig file.

Usually located at:

            ~/.kube/config

Inside you'll see something like:

      users:
      - name: kubernetes-admin
        user:
          client-certificate: /etc/kubernetes/pki/admin.crt
          client-key: /etc/kubernetes/pki/admin.key

So the flow is:

            kubectl
            
            ↓
            
            ~/.kube/config
            
            ↓
            
            admin.crt
            
            ↓
            
            API Server



Step 9: How Does the API Server Identify You?
----------------------------------------------
Suppose the certificate contains:

            CN = aravind
            
            O = developers

Kubernetes interprets this as:
            
            User = aravind
            
            Group = developers

RBAC can then grant permissions to:

            User aravind
            Group developers

Step 10: How the API Server Verifies the Certificate
----------------------------------------------------------
When a request arrives, the API Server checks:

            Client Certificate
                    │
                    ▼
            1. Is it signed by my CA?
                    │
            2. Is it expired?
                    │
            3. Is the signature valid?
                    │
            4. Does the private key match?
                    │
                    ▼
            Authentication Success

If any of these checks fail:

Authentication Failed
----------------------
The request never reaches RBAC.

Step 11: Commands Every DevOps Engineer Should Know
---------------------------------------------------------
View certificates:

            ls /etc/kubernetes/pki

View certificate details:

            openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout

Check expiry:

            openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -dates

View kubeconfig:

      kubectl config view

Current context:

      kubectl config current-context

🎤 Interview Questions
--------------------------
1. Why does Kubernetes use certificates?

Answer: Kubernetes uses TLS certificates to securely authenticate users and cluster components.
Certificates verify identity and encrypt communication between clients and the API server.

2. What is the role of the Kubernetes CA?

Answer: The Kubernetes Certificate Authority (CA) signs certificates for the API server, kubelets, and clients.
The API server trusts only certificates signed by this CA.


3. Where does kubectl get its authentication credentials?

Answer: kubectl reads the kubeconfig file (typically ~/.kube/config), 
which contains cluster information and authentication details such as a client certificate, client key, or a token.

🔑 One Important Point to Remember
------------------------------------
There are two types of authentication you'll encounter in Kubernetes:

Certificate-based authentication (mostly for humans and cluster components)
kubectl
Administrators
Kubelets
Control plane components
Token-based authentication (mostly for applications running inside the cluster)
ServiceAccounts
Pods
IRSA in EKS

This distinction is why learning certificates first makes understanding ServiceAccounts and IRSA much easier.






