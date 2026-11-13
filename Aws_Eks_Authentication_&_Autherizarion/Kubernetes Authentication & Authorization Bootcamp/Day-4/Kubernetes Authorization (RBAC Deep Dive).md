Kubernetes Authorization (RBAC Deep Dive)
-----------------------------------------

🎯 Goal

By the end of this session, you'll understand:

How Kubernetes authorizes every request
How RBAC evaluates permissions
How kubectl auth can-i works internally
Why you get Forbidden errors
How to troubleshoot RBAC issues in production
Advanced RBAC concepts used in enterprises

Step 1: Complete Request Flow
------------------------------
Let's put together everything we've learned so far.

Suppose you execute:
kubectl get pods -n dev


The request follows this path:

                     kubectl
                        │
                        ▼
              API Server Receives Request
                        │
                        ▼
              Authentication Phase
                        │
              (Certificate / Token / IAM / OIDC)
                        │
            Is the identity valid?
                │              │
               YES             NO
                │              │
                ▼              ▼
     Authorization Phase   Reject Request
            (RBAC)
              │
              ▼
     Does the user have permission?
              │            │
             YES          NO
              │            │
              ▼            ▼
     Execute API     Forbidden Error

Important interview point:

Authentication answers: Who are you?

Authorization answers: What are you allowed to do?

Step 2: How RBAC Makes Decisions
-----------------------------------

Suppose this authenticated identity reaches the API Server:

User:
system:serviceaccount:dev:developer

Request:

kubectl get pods -n dev

RBAC evaluates:
Internal Decision Tree
      Request
      
      ↓
      
      Identity
      
      ↓
      
      RoleBinding?
      
      ↓
      
      Role?
      
      ↓
      
      Correct Namespace?
      
      ↓
      
      Correct API Group?
      
      ↓
      
      Correct Resource?
      
      ↓
      
      Correct Verb?
      
      ↓
      
      ALLOW

If any check fails, Kubernetes returns:

      Forbidden

Question 1

Is there any RoleBinding?

YES

↓

Question 2

Which Role is attached?

developer-role

↓

Question 3

Does that Role contain:

Resource = pods

↓

YES

↓

Question 4

Does it allow:

Verb = get

↓

YES

↓

ALLOW



If any check fails, Kubernetes returns:

Forbidden

Step 3: Understanding API Groups
---------------------------------
Many RBAC problems happen because people forget the API group.

Examples:

Pods

          apiGroups:
          - ""
          Core API group.

Deployments:

          apiGroups:
          - apps

- 
Jobs

              apiGroups:
              - batch
  
Ingress

              apiGroups:
              - networking.k8s.io

If you specify the wrong API group, RBAC will not match the request.

Step 4: Verbs
-------------
Kubernetes permissions are based on verbs.

Common verbs:
| Verb             | Meaning                   |
| ---------------- | ------------------------- |
| get              | Read one object           |
| list             | List multiple objects     |
| watch            | Watch for changes         |
| create           | Create resources          |
| update           | Replace existing resource |
| patch            | Modify part of a resource |
| delete           | Delete resource           |
| deletecollection | Delete multiple resources |

Example:

verbs:
- get
- list

Can the user create Pods?

No.

Because create isn't allowed.

Step 5: kubectl auth can-i
---------------------------
This is the best RBAC troubleshooting command.

Example:

kubectl auth can-i get pods

Output:

yes

Check another identity:

kubectl auth can-i get pods \
--as=system:serviceaccount:dev:developer \
-n dev

Expected:

yes

Delete?

kubectl auth can-i delete pods \
--as=system:serviceaccount:dev:developer \
-n dev

Expected:

no
What does --as do?

Imagine you're the Kubernetes administrator.

You want to know:

"Can this ServiceAccount delete Pods?"

Instead of logging in as that ServiceAccount, you ask the API server:

"Evaluate this request AS IF I were this identity."

That's exactly what:

--as=system:serviceaccount:dev:developer

does.

It is called impersonation.

Step 6: Multiple Roles
------------------------
Suppose:

Role A

      Resources:
      Pods

Role B

      Resources:
      ConfigMaps

Both are bound to the same ServiceAccount.

Developer

↓

RoleBinding A

↓

Role A

+

RoleBinding B

↓

Role B

Effective permissions become:

Pods

+

ConfigMaps

RBAC permissions are additive.

Step 7: There is NO Explicit Deny
---------------------------------
This is different from AWS IAM.

AWS IAM has:

Allow
Explicit Deny

Kubernetes RBAC has:

Allow
No matching rule = Deny

Example:

Role A

get Pods

Role B

create Pods

Effective permissions:

get Pods

create Pods

You cannot write a rule that says:

deny:
- delete

RBAC simply doesn't support explicit deny.

Step 8: Production Troubleshooting
-------------------------------------
Suppose a developer reports:

"I can't read ConfigMaps."

Step 1

Check:

kubectl auth can-i get configmaps \
--as=system:serviceaccount:dev:developer \
-n dev

Output:

no
Step 2

Inspect RoleBindings

kubectl get rolebinding -n dev
Step 3

Describe RoleBinding

kubectl describe rolebinding developer-binding -n dev

Verify:

Correct ServiceAccount?
Correct Role?
Step 4

Describe Role

kubectl describe role developer-role -n dev

Suppose:

resources:
- pods

verbs:
- get

ConfigMaps are missing.

Step 5

Update Role

resources:
- pods
- configmaps

verbs:
- get
- list

Apply:

kubectl apply -f role.yaml

Verify again:

kubectl auth can-i get configmaps \
--as=system:serviceaccount:dev:developer \
-n dev

Now:

yes
Step 9: Enterprise RBAC Design
--------------------------------
Imagine a company with three teams.

Developers

Namespace:

dev

Permissions:

Pods
Deployments
Services
ConfigMaps

No access to production.

QA

Namespace:

qa

Permissions:

Pods
Logs
Restart Deployments
Platform Team

Cluster-wide access:

Nodes
StorageClasses
Namespaces
CRDs
ClusterRoles
Security Team

Cluster-wide read access plus Secrets management.

Step 10: Useful Debug Commands
-------------------------------
View Roles:

kubectl get roles -A

View ClusterRoles:

kubectl get clusterroles

View RoleBindings:

kubectl get rolebindings -A

View ClusterRoleBindings:

kubectl get clusterrolebindings

Describe Role:

kubectl describe role developer-role -n dev

Describe ClusterRole:

kubectl describe clusterrole node-reader

🧪 Hands-on Lab 1
------------------------------
Create:

Namespace: dev
ServiceAccount: developer
Role: pod-reader
RoleBinding

Role permissions:

resources:
- pods

verbs:
- get
- list

Verify:

kubectl auth can-i get pods \
--as=system:serviceaccount:dev:developer \
-n dev

Expected:

yes
🧪 Hands-on Lab 2
--------------------
Create another Role for:

resources:
- configmaps

verbs:
- get
- list

Bind it to the same ServiceAccount.

Verify:

kubectl auth can-i get configmaps \
--as=system:serviceaccount:dev:developer \
-n dev

Then verify that the same ServiceAccount can still access Pods.

🧪 Hands-on Lab 3 (Troubleshooting)
----------------------------------------
Create a Role that allows only get on Pods.
Try:
kubectl auth can-i delete pods \
--as=system:serviceaccount:dev:developer \
-n dev

Expected:

no
Update the Role to include:
verbs:
- get
- delete
Apply the updated Role.
Verify again.

🎤 Interview Questions
-------------------------
Q1. What happens after Kubernetes authenticates a user?

Answer: After successful authentication, the API server evaluates RBAC. It checks RoleBindings and ClusterRoleBindings associated with the authenticated identity, then matches the requested resource, verb, API group, and namespace against the rules in the referenced Role or ClusterRole. If a matching rule exists, the request is allowed; otherwise, it is denied.

Q2. What is the purpose of kubectl auth can-i?

Answer: It checks whether a user or ServiceAccount has permission to perform a specific action on a resource. It is commonly used to troubleshoot RBAC issues without executing the actual operation.

Q3. Can a ServiceAccount have multiple RoleBindings?

Answer: Yes. A ServiceAccount can be associated with multiple RoleBindings and ClusterRoleBindings. Kubernetes combines all the allowed permissions, so the effective permissions are the union of all granted rules.

