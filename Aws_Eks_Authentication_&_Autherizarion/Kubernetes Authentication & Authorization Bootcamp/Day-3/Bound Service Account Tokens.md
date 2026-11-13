Topic 2: Bound Service Account Tokens
=====================================

Learning Objectives
-------------------
By the end of this topic, you'll understand:

* Why Bound Service Account Tokens were introduced
* How Pods automatically receive tokens
* Where the token is stored inside a Pod
* How Kubernetes automatically rotates (refreshes) the token
* Difference between old tokens and bound tokens
* How this forms the foundation of IRSA

First, let's understand the problem.

Suppose we have a ServiceAccount:

      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: app-sa

And a Pod:

    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      serviceAccountName: app-sa


Question:

How does this Pod authenticate to the Kubernetes API Server?

The Pod cannot simply say:

Hi API Server,
I'm app-sa.
Trust me.

The API Server won't trust it.

The Pod needs proof.

That proof is a JWT token.

Before Kubernetes v1.24
------------------------
When you created the ServiceAccount:

    ServiceAccount
    
    ↓
    
    Kubernetes automatically created Secret
    
    ↓
    
    Permanent JWT Token
    
    ↓
    
    Pod mounted Secret

Example:

    ServiceAccount
          │
          ▼
    Secret
    app-sa-token-ab123
    
          │
          ▼
    Mounted inside Pod

Problems:

    Token never expired
    Token stored as Secret
    Easy to steal
    No automatic rotation

Kubernetes Improved This
-------------------------
Instead of creating a Secret, Kubernetes now creates a Bound Service Account Token.

The flow is:

    Pod Created
        ↓
    Kubelet
        ↓
    TokenRequest API
        ↓
    API Server
        ↓
    Short-lived JWT
        ↓
    Kubelet
        ↓
    Mounted into Pod
        ↓
    Token Rotation


1. Pod is created

Suppose you create this Pod:
    
    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp
    spec:
      serviceAccountName: myapp-sa
      containers:
      - name: app
        image: nginx

You are saying:

"Run this Pod using the myapp-sa ServiceAccount."

So the relationship is:

                  Pod
                   ↓
                  myapp-sa

2. Kubelet gets involved

The Pod is scheduled to a worker node.

For example:

        Kubernetes Cluster
        
        
        Worker Node
            ↓
        Kubelet
            ↓
        Pod: myapp

The kubelet is responsible for managing the Pod on that worker node.

It sees:

      Pod → ServiceAccount = myapp-sa

and knows that the Pod needs a ServiceAccount token.

3. Kubelet requests a token

The kubelet communicates with the Kubernetes API server.

It uses the TokenRequest API.

Conceptually:

            Kubelet
               |
               | "Give me a token for myapp-sa
               |  for this Pod"
               ↓
            API Server

The request contains information such as:

            ServiceAccount = myapp-sa
            Namespace = dev
            Pod identity
            Requested expiration

4. API Server creates the JWT

The API server validates the request.

If everything is valid, Kubernetes creates a short-lived JWT.

For example:

            JWT
            │
            ├── ServiceAccount identity
            ├── Namespace
            ├── Pod information
            ├── Issuer
            └── Expiration time

Think of this token as:

A temporary identity card for the Pod.

For example:

            Token valid:
            10:00 AM → 11:00 AM

The exact lifetime depends on the cluster's configuration and limits.

5. API Server gives the token back to kubelet

The API server doesn't directly put the token inside the container.

Instead:

            API Server
                ↓
            JWT
                ↓
            Kubelet

The kubelet receives the token.

6. Kubelet mounts the token into the Pod

Now the kubelet makes the token available inside the Pod.

Typically you'll find it here:

    /var/run/secrets/kubernetes.io/serviceaccount/token

Inside the container:

    cat /var/run/secrets/kubernetes.io/serviceaccount/token

You would see something like:

    eyJhbGciOiJSUzI1NiIs...

That's the JWT.

The Pod also normally gets:

    /var/run/secrets/kubernetes.io/serviceaccount/
    │
    ├── token
    ├── ca.crt
    └── namespace

So your application can use the token when communicating with the Kubernetes API server.


7. Application uses the token

Suppose your application inside the Pod wants to call:

Kubernetes API Server

It can use:

    /var/run/secrets/kubernetes.io/serviceaccount/token

The request looks conceptually like:

      Application
          ↓
      JWT Token
          ↓
      API Server

The API server checks:

    Who are you?
            ↓
    ServiceAccount: myapp-sa
    
    
    What are you allowed to do?
            ↓
    RBAC

Then Kubernetes decides whether to allow the request.

8. Token expires

Now suppose the token was valid for 1 hour:

    10:00
      ↓
    Token issued
    
    
    10:30
      ↓
    Still valid
    
    
    10:55
      ↓
    Kubelet prepares replacement
    
    
    11:00
      ↓
    Old token expires

The kubelet doesn't simply wait until the token becomes invalid.

It requests a new token before expiration.

      Old Token
          ↓
      Almost expired
          ↓
      Kubelet
          ↓
      TokenRequest API
          ↓
      API Server
          ↓
      New JWT
          ↓
      Kubelet
          ↓
      Token gets rotated

This is called token rotation.

Complete flow

Put everything together:

                                  Kubernetes API Server
                                           ↑
                                           |
                                     TokenRequest
                                           |
                                           |
                  Pod Created → Kubelet ───┘
                                  |
                                  | receives JWT
                                  ↓
                          Short-lived JWT
                                  |
                                  ↓
                        Mounted into Pod
                                  |
                                  ↓
                  /var/run/secrets/kubernetes.io/serviceaccount/token
                                  |
                                  ↓
                         Application uses JWT
                                  |
                                  ↓
                          API Server validates
                                  |
                                  ↓
                               RBAC
                                  |
                                  ↓
                          Allow / Deny



And when the token is close to expiration:

      Token nearly expires
              ↓
      Kubelet requests new token
              ↓
      TokenRequest API
              ↓
      API Server
              ↓
      New JWT
              ↓
      Token rotated
              ↓
      Application continues working


The 4 things I want you to remember

        1. Kubelet requests the token — not the application directly.
        2. API Server issues the short-lived JWT.
        3. Kubelet makes the token available inside the Pod.
        4. Kubelet automatically rotates the token before expiration.

And importantly, this is why modern Kubernetes doesn't need to create a permanent myapp-sa-token-xxxxx Secret for every ServiceAccount.


Notice something important.

The token is created because the Pod needs it.

It is bound to that specific Pod.

This is why it's called a Bound Service Account Token.

What does "Bound" mean?
-----------------------

This is the most common interview question.

Imagine two Pods.

Pod A
↓

ServiceAccount app-sa

and

Pod B

↓

ServiceAccount app-sa

Even though both use the same ServiceAccount,

they do not share the same token.

Instead:

Pod A

↓

JWT Token A


Pod B

↓

JWT Token B

Each Pod gets its own token.

That token is bound to that Pod.

If Pod A is deleted,

its token becomes invalid.

Complete Authentication Flow
------------------------------
Let's understand everything happening internally.

Suppose you apply:

    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      serviceAccountName: app-sa

Internally Kubernetes performs:

Step 1

Pod Created

↓

Scheduler

Schedules Pod

↓

Kubelet

Starts Pod

↓

Kubelet notices

This Pod uses ServiceAccount app-sa

↓

Kubelet

Requests Token

↓

POST

TokenRequest API

↓

API Server

Generates JWT

↓

Returns JWT

↓

Kubelet mounts token

Inside Pod

↓

Application reads token

Authenticates to API Server

Where is the token stored?
--------------------------
Inside every Pod:

            /var/run/secrets/kubernetes.io/serviceaccount/

Let's verify.

Create Pod:

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  serviceAccountName: app-sa

  containers:
  - name: nginx
    image: nginx

Login to Pod

kubectl exec -it nginx -- sh

Check:

            cd /var/run/secrets/kubernetes.io/serviceaccount

List files:

            ls

Output

                  ca.crt
                  namespace
                  token

Read token

cat token

You'll see

eyJhbGciOiJSUzI1Ni...

That is the JWT.

What are these files?
-----------------------
token

JWT token

Used to authenticate.

namespace

Contains

default

or

dev

So the application knows which namespace it is running in.

ca.crt

Certificate Authority certificate.

The application uses it to verify that it is really talking to the Kubernetes API Server and not an imposter.

Without ca.crt, the application cannot securely establish HTTPS communication with the API Server.

Automatic Token Rotation
-------------------------
Another major improvement.

Old token

Created

↓

Never Changed

New token

Created

↓

Expires

↓

Kubelet requests new token

↓

Replaces old token

↓

Application keeps working

Kubernetes refreshes the token before it expires.

No restart required.

Real Production Example

Imagine Prometheus running inside Kubernetes.

Prometheus

↓

Needs Pod Metrics

↓

Calls Kubernetes API

Every request includes:

Authorization:

Bearer <JWT Token>

If the token expires,

Kubelet gets a new one.

Prometheus continues working.

No manual action.

Hands-on Lab
Step 1

Create ServiceAccount

kubectl create sa app-sa
Step 2

Create Pod

apiVersion: v1
kind: Pod
metadata:
  name: nginx

spec:
  serviceAccountName: app-sa

  containers:
  - name: nginx
    image: nginx

Apply

kubectl apply -f pod.yaml
Step 3

Login

kubectl exec -it nginx -- sh
Step 4

Go to token directory

cd /var/run/secrets/kubernetes.io/serviceaccount
Step 5

List files

ls

Output

ca.crt
namespace
token
Step 6

Read namespace

cat namespace
Step 7

Read token

cat token

Old vs New
------------
Feature	Old ServiceAccount Token	Bound ServiceAccount Token
| Feature               | Old ServiceAccount Token | Bound ServiceAccount Token |
| --------------------- | ------------------------ | -------------------------- |
| Stored as Secret      | ✅ Yes                    | ❌ No                       |
| Permanent             | ✅ Yes                    | ❌ No                       |
| Expires               | ❌ Never                  | ✅ Yes                      |
| Auto Rotation         | ❌ No                     | ✅ Yes                      |
| Bound to Pod          | ❌ No                     | ✅ Yes                      |
| Uses TokenRequest API | ❌ No                     | ✅ Yes                      |

Real Company Scenario (EKS)
---------------------------
Suppose your application needs to:

Read Pods
Read Services
Read ConfigMaps

Instead of storing a permanent token inside the container image (which is insecure), Kubernetes automatically mounts a Bound Service Account Token into the Pod. The application reads that token whenever it calls the Kubernetes API.

The same mechanism is later used by IRSA to obtain temporary AWS credentials without storing AWS Access Keys in the Pod.

Interview Questions
--------------------
1. What is a Bound Service Account Token?

A Bound Service Account Token is a short-lived JWT generated by the Kubernetes API Server using the TokenRequest API. It is mounted into a Pod, automatically refreshed before expiration, and is bound to that Pod, making it more secure than the older long-lived ServiceAccount token Secrets.

2. Why is it called a "Bound" token?

Because the token is issued specifically for a Pod using a particular ServiceAccount. If that Pod is deleted, the token becomes invalid. Kubernetes does not create one permanent token shared across all Pods.

3. Where is the token mounted inside the Pod?
/var/run/secrets/kubernetes.io/serviceaccount/token
4. Who requests the token for the Pod?

The Kubelet requests a short-lived token from the Kubernetes API Server using the TokenRequest API and mounts it into the Pod as a projected volume.
