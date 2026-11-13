Module 11: Debugging 403 Forbidden
=======================================
Suppose a developer runs:

kubectl get pods

and gets:

Error from server (Forbidden)

Follow this checklist:

Step 1: Verify the current identity
--------------------------------------
kubectl config current-context

Step 2: Check permissions
----------------------------
kubectl auth can-i get pods

Example:

yes

or

Step 3: Test another action
-----------------------------
kubectl auth can-i create deployment

Step 4: Test as a specific ServiceAccount
------------------------------------------
kubectl auth can-i list pods \
--as=system:serviceaccount:cicd:jenkins


Step 5: Check RoleBindings
-----------------------------
kubectl get rolebinding -A

Step 6: Check ClusterRoleBindings
----------------------------------
kubectl get clusterrolebinding

Step 7: Describe the binding
-----------------------------
kubectl describe rolebinding developer

Verify:

Subject
Namespace
RoleRef
