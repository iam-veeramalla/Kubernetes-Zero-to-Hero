What is Dex?
================
Dex is an OIDC Identity Provider commonly used with Kubernetes.

It can connect to many identity backends:

LDAP
Active Directory
GitHub
GitLab
SAML
Google
Microsoft

Architecture:
---------------
kubectl

↓

Dex

↓

LDAP

↓

JWT Token

↓

API Server

↓

RBAC

Dex translates identities from your existing directory into OIDC tokens that Kubernetes understands.
