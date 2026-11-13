Why Kubernetes Doesn't Store Users
======================================

Unlike ServiceAccounts...

Kubernetes has:

Pods
Deployments
Nodes
Secrets
ConfigMaps
ServiceAccounts

But...

There is NO User object.

Why?

Because companies already have employee identity systems.

Example:

Company Employees

John
Rahul
David
Mike
Priya

These users already exist in

1. LDAP
2. Active Director
3. Azure AD
4. Okta
5. AWS IAM Identity Center

Kubernetes simply trusts those systems.


Production Example

Employee joins company

↓

HR creates account

↓

Account automatically appears in

Active Directory

↓

Employee logs into laptop

↓

Employee logs into Kubernetes

↓

Kubernetes trusts Active Directory

No user creation inside Kubernetes.
