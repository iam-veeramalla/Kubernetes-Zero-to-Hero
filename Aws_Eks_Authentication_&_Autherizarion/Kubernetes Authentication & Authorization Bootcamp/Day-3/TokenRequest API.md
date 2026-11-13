Topic 1: TokenRequest API
===========================

This topic is the foundation for understanding Bound Service Account Tokens and IRSA.

Why was the TokenRequest API introduced?

Let's first understand the old behavior.

Before Kubernetes v1.24

Whenever you created a ServiceAccount:

      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: app-sa

Kubernetes automatically created a Secret containing a ServiceAccount token.

        ServiceAccount
              │
              ▼
        +----------------------------+
        | app-sa-token-x7m2k         |
        |----------------------------|
        | token                      |
        | ca.crt                     |
        | namespace                  |
        +----------------------------+

You could verify it with:
 
   kubectl get secrets


Example output:

      NAME                    TYPE
      app-sa-token-x7m2k      kubernetes.io/service-account-token

Problem with Old Tokens
---------------------------
These tokens had several security issues.

Problem 1: 

    Never Expired: The token remained valid indefinitely.If someone stole it, they could continue using it until it was manually revoked.


Created Today

↓

Still valid after

1 day
30 days
1 year
5 years




Problem 2: 

      Stored as a Secret: The token was stored in etcd as a Secret.Anyone with permission to read that Secret could authenticate as the ServiceAccount.


Problem 3: 

    Easy to Leak: 

          Developers sometimes copied the token into:
          
          Git repositories ❌
          CI/CD variables ❌
          Documentation ❌
          Slack messages ❌

Since the token never expired, it created a long-term security risk.

Kubernetes Solution
-------------------

Kubernetes introduced the TokenRequest API.

Instead of storing a permanent token, the API Server now creates short-lived tokens on demand.


      ServiceAccount
      
      ↓
      
      TokenRequest API
      
      ↓
      
      Temporary JWT Token
      
      ↓
      
      Expires Automatically


This is much more secure.

How to Create a Token
-------------------------

Create a ServiceAccount:

      kubectl create serviceaccount demo-sa

Now request a token:

      kubectl create token demo-sa

Example output:

      eyJhbGciOiJSUzI1NiIsImtpZCI6...

This is a JWT (JSON Web Token).

Unlike the old tokens, it has an expiration time.


Specify Token Lifetime
------------------------

You can request a custom duration:

      kubectl create token demo-sa --duration=30m

or

      kubectl create token demo-sa --duration=2h

Example:

      Requested
      
      ↓
      
      30 minutes
      
      ↓
      
      Automatically expires

Note: 

     The API server may enforce a maximum lifetime. If you request a longer duration than allowed, it can return a shorter-lived token.
      Means: But the Kubernetes API server has a maximum allowed token lifetime configured by the cluster administrator.

Where is the Token Stored?

Unlike the old mechanism:

      ServiceAccount
      
      ↓
      
      API Server
      
      ↓
      
      Returns Token
      
      ↓
      
      No Secret Created

kubectl get secrets

On Kubernetes v1.24+ you typically won't see a ServiceAccount token Secret created automatically.

So where is the token stored?

This is the important part.

When a Pod uses a ServiceAccount, Kubernetes makes the token available inside the Pod's filesystem.

Typically:

      /var/run/secrets/kubernetes.io/serviceaccount/token

For example:

      kubectl exec -it mypod -- cat /var/run/secrets/kubernetes.io/serviceaccount/token

You will see the JWT token.

There are usually three important files:

      /var/run/secrets/kubernetes.io/serviceaccount/
      │
      ├── token
      ├── ca.crt
      └── namespace

Real Production Example
-------------------------

Suppose a monitoring application runs in Kubernetes.

      Monitoring Pod
      
      ↓
      
      Needs to read Pods
      
      ↓
      
      Uses ServiceAccount
      
      ↓
      
      Requests a temporary token
      
      ↓
      
      Calls Kubernetes API
      
      ↓
      
      Token expires automatically

Even if an attacker steals the token, it becomes useless after it expires.

Hands-on Lab
-------------
Step 1: Create a namespace
kubectl create namespace demo
Step 2: Create a ServiceAccount
kubectl create serviceaccount demo-sa -n demo
Step 3: Verify
kubectl get sa -n demo

Expected:

NAME      SECRETS   AGE
demo-sa   0         10s

Notice the SECRETS column is 0. This shows Kubernetes did not automatically create a long-lived token Secret.

Step 4: Request a token
kubectl create token demo-sa -n demo

A JWT token will be returned.

Interview Questions
---------------------
1. Why was the TokenRequest API introduced?

To replace long-lived ServiceAccount tokens with short-lived, automatically expiring tokens, reducing the risk of credential theft.

2. What changed in Kubernetes v1.24?

Kubernetes stopped automatically creating long-lived ServiceAccount token Secrets. Tokens are now issued dynamically through the TokenRequest API.

3. Does kubectl create token create a Secret?

No. It requests a temporary token from the API Server. No ServiceAccount token Secret is created.



