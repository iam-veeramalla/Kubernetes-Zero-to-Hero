Kubernetes Authentication Using LDAP
=======================================

Flow
----
kubectl

↓

API Server

↓

LDAP Server

↓

Validate username/password

↓

Success

↓

RBAC Authorization

Notice something important.

Authentication happens first.

Authorization happens later.

Exactly what we learned in previous days.
