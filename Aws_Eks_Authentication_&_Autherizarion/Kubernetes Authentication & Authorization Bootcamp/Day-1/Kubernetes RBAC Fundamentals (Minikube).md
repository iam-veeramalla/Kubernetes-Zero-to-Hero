Day 1: Kubernetes RBAC Fundamentals (Minikube)
----------------------------------------------

Goal: Understand how Kubernetes decides who can do what.

By the end of today, you'll be able to answer interview questions like:

    1. What is RBAC?
    2. What is the difference between Role and ClusterRole?
    3. What is RoleBinding?
    4. How does Kubernetes authorization work?
    5. How do you verify permissions?

Today's Architecture: 

                  kubectl get pods
                         │
                         ▼
               Kubernetes API Server
                         │
          Authentication (Who are you?)
                         │
                         ▼
        Authorization (RBAC: What can you do?)
                         │
                  Allowed / Denied

Today we'll focus only on the Authorization (RBAC) part.

===================================================================================================================================================================

Lab Prerequisites
-----------------
Start Minikube: minikube start --driver=docker --force


Verify:kubectl get nodes

Step 1: See who you are
=========================

Run:

    kubectl config current-context

result : minikube


Now check:

    kubectl config view --minify


Look for:users:
- name: minikube

- Explanation

Right now you are connecting as the minikube user, which has cluster-admin privileges.


Step 2: Verify your permissions:



Run:

    kubectl auth can-i create pods----> yes

Run: 

    kubectl auth can-i delete namespaces--> yes

Question: Why do you think both commands return yes?

Because the default Minikube user has administrative permissions.


Step 3: Create a Namespace: 

    kubectl create namespace dev --> to create a ns

    kubectl get ns ---> to see the list of ns


Step 4: Create a Role:


Create a file named developer-role.yaml.

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    
    metadata:
      name: developer-role
      namespace: dev
    
    rules:
    - apiGroups: [""]
      resources:
        - pods
        - services
      verbs:
        - get
        - list
        - watch
        - create
     
kubectl apply -f developer-role.yaml 

kubectl get role -n dev 

Understanding the Role: 


    resources:
    - pods
    - services

    verbs:
    - get
    - list
    - watch
    - create

This role allows:

    | Action           | Allowed |
    | ---------------- | ------- |
    | Get Pods         | ✅       |
    | List Pods        | ✅       |
    | Watch Pods       | ✅       |
    | Create Pods      | ✅       |
    | Delete Pods      | ❌       |
    | Delete Namespace | ❌       |
    | View Secrets     | ❌       |

A Role is always limited to a single namespace (dev in this case).

Step 5: Create a Service Account:

* Instead of creating certificates for a user today, 
* we'll use a ServiceAccount because it's the easiest way to demonstrate RBAC.

Run: 

    kubectl create serviceaccount developer -n dev


    kubectl get sa -n dev

Step 6: Bind the Role:


Create developer-rolebinding.yaml.


    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    
    metadata:
      name: developer-binding
      namespace: dev
    
    subjects:
    - kind: ServiceAccount
      name: developer
      namespace: dev
    
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: Role
      name: developer-role

Apply it: 

    kubectl apply -f developer-rolebinding.yaml
  
    kubectl get rolebinding -n dev


Step 7: Test Permissions:

  Without switching users, Kubernetes lets you check permissions using impersonation.

  Check if the developer service account can create pods:
   
Run:

     kubectl auth can-i create pods \
    --as=system:serviceaccount:dev:developer \
    -n dev
  
    kubectl auth can-i delete pods \
    --as=system:serviceaccount:dev:developer \
    -n dev


* This is the easiest way to verify RBAC rules without creating kubeconfigs for different users.

What You've Learned Today

        Role
          │
          ▼
    Defines Permissions
          │
          ▼
    RoleBinding
          │
          ▼
    Assigns Role
          │
          ▼
    ServiceAccount (or User/Group)




===========================================

Interview Questions
1. What is RBAC?

Answer:

RBAC (Role-Based Access Control) is Kubernetes' authorization mechanism. It controls what authenticated users, groups, or service accounts are allowed to do within the cluster.

2. What is a Role?

Answer:

A Role defines a set of permissions within a specific namespace.

3. What is a RoleBinding?

Answer:

A RoleBinding assigns a Role to a user, group, or service account within the same namespace.

4. Why did we use a ServiceAccount?

Answer:

ServiceAccounts are Kubernetes identities used by workloads running inside the cluster. 
They are also convenient for demonstrating RBAC in a lab. In production, 
human users are more commonly authenticated through external identity providers (such as AWS IAM, OIDC, or Active Directory), while applications use ServiceAccounts.

Homework
Create a namespace named qa.
Create a qa-role that allows only:
get
list
Create a ServiceAccount named qa-user.
Create a RoleBinding.
Verify:
kubectl auth can-i get pods → yes
kubectl auth can-i create pods → no
One important note

You may wonder why we're using a ServiceAccount instead of a real user
. That's intentional. Kubernetes itself does not manage human users. In production:

Minikube: Human users are usually authenticated with client certificates.
kOps: Human users are often authenticated via AWS IAM Authenticator.
EKS: Human users are commonly authenticated via IAM Identity Center + EKS Access Entries.

By learning RBAC first, you'll understand the authorization layer that works the same across all of these authentication methods.
On Day 2, we'll build on this by covering ClusterRoles, ClusterRoleBindings, and the difference between namespace-scoped and cluster-wide permissions.
