Kubernetes Users, Authentication Flow & kubeconfig
==================================================

Goal: By the end of today, you'll be able to explain exactly what happens behind the scenes when someone runs:

kubectl get pods

This is one of the most frequently asked interview topics for Kubernetes, EKS, and DevOps roles.

Today's Learning Objectives
----------------------------
By the end of Day 3, you'll understand:

Kubernetes Users vs ServiceAccounts
How Kubernetes Authentication works
What is kubeconfig?
Authentication Flow
Authorization Flow
Complete request lifecycle
Hands-on
Interview Questions

Today we'll answer a different question.
-----------------------------------------
Question
--------
When you run:

kubectl get pods

How does Kubernetes know:

Who you are?
Which cluster to connect to?
Whether you're allowed to list pods?

Today's lesson answers all three.

Part 1 - Kubernetes User vs ServiceAccount
-------------------------------------------
This confuses many beginners.

Kubernetes User

A Kubernetes User represents a human or an external system.

Examples:
---------
DevOps Engineer
Developer
Jenkins
GitHub Actions

Example:
-------
Aravind
    │
kubectl
    │
API Server
Important

Unlike ServiceAccounts:

Users are NOT stored inside Kubernetes.

Run:

kubectl get users

You'll get an error because there is no User object in Kubernetes.

ServiceAccount
---------------
ServiceAccounts represent applications running inside the cluster.

Example:

Pod
 │
ServiceAccount
 │
API Server

Examples:

Prometheus
Argo CD
NGINX Ingress Controller
Your Java application
Interview Question
Does Kubernetes have a User object?

Answer:

No.

Users exist outside Kubernetes and are authenticated using certificates, tokens, or external identity providers.

Part 2 - What is kubeconfig?
------------------------------
When you run:

kubectl get pods

How does kubectl know:

Cluster endpoint?
User?
Namespace?
Certificate?

Answer:

~/.kube/config

On Windows:

C:\Users\<username>\.kube\config

Part 3 - Let's View It
---------------------------
Run:

kubectl config view

You'll see sections like:

clusters:
users:
contexts:
current-context:

Let's understand each.

Clusters

Example:

clusters:
- cluster:
    server: https://XXXXXXXX.gr7.ap-south-1.eks.amazonaws.com

This tells kubectl:

Connect to this API Server.

Users

Example:

users:
- name: dev-user

This defines how to authenticate.

In EKS, you'll usually see an exec section that calls the AWS CLI to generate a temporary authentication token.

Context

A context combines:

Cluster
User
Namespace

Example:

context:
  cluster: production
  user: dev-user
  namespace: default

Think of a context as a profile.

Current Context

Run:

kubectl config current-context

This tells you which context kubectl is currently using.

Part 4 - Authentication Flow
------------------------------
Let's trace the full request.

You run:

kubectl get pods

The flow is:

kubectl
   │
Reads kubeconfig
   │
Gets Cluster + User
   │
Sends request
   │
Kubernetes API Server

The API Server's first question is:

Who are you?

This is Authentication.

If authentication fails:

401 Unauthorized

Part 5 - Authorization
------------------------
If authentication succeeds:

The API Server asks:

What are you allowed to do?

Now RBAC checks:

Role
RoleBinding
ClusterRole
ClusterRoleBinding

If permission is denied:

403 Forbidden
Part 6 - Complete Request Lifecycle
Developer

↓

kubectl get pods

↓

Read kubeconfig

↓

Connect to API Server

↓

Authentication
(Who are you?)

↓

Authorization
(What can you do?)

↓

Admission Controllers
(Any policies?)

↓

etcd

↓

Response

↓

kubectl output

This entire flow happens in milliseconds.

Real EKS Example
------------------
Suppose you run:

aws eks update-kubeconfig \
  --name production-cluster \
  --region ap-south-1

This command updates your kubeconfig with:

Cluster endpoint
Certificate authority
Authentication method
Context

Later, when you run:

kubectl get nodes

kubectl uses that information automatically.

Hands-on
--------
Step 1

View your kubeconfig:

kubectl config view
Step 2

List contexts:

kubectl config get-contexts
Step 3

Current context:

kubectl config current-context
Step 4

Display only the active configuration:

kubectl config view --minify

Identify:

Current cluster
Current user
Current namespace
Step 5

Check your permissions:

kubectl auth can-i get pods

Now try:

kubectl auth can-i delete pods

Observe whether your current identity has permission.

Interview Questions
-------------------
Answer these in your own words after the hands-on:

What is the difference between a Kubernetes User and a ServiceAccount?
Does Kubernetes have a User object? Explain.
What is the purpose of the kubeconfig file?
What are the four main sections of a kubeconfig file?
Explain what happens when you run kubectl get pods.
What is the difference between Authentication and Authorization?
What is the difference between a 401 Unauthorized error and a 403 Forbidden error?


Tomorrow (Day 4 Preview)
--------------------------
We'll build on today's authentication concepts by covering:

ServiceAccount Tokens
TokenRequest API
Bound Service Account Tokens
How Pods authenticate to the Kubernetes API
Why Kubernetes moved away from long-lived ServiceAccount secrets
How these concepts lead directly into IAM Roles for Service Accounts (IRSA) in Amazon EKS.

This sequence is designed so each day builds naturally on the previous one, especially if your goal is to become strong in Kubernetes security and AWS EKS.
