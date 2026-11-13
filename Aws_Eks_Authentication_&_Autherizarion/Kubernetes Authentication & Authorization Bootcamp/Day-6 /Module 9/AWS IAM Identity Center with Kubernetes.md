AWS IAM Identity Center with Kubernetes
========================================
For Amazon EKS, many organizations use AWS IAM Identity Center (formerly AWS SSO).

Flow:

Developer

↓

AWS IAM Identity Center Login

↓

MFA

↓

OIDC Identity

↓

AWS CLI Authentication

↓

kubectl

↓

EKS API Server

↓

AWS IAM Authenticator

↓

RBAC Groups

↓

Cluster Access

Benefits:

Centralized user management
MFA
Short-lived credentials
Easy onboarding/offboarding
Integrates with AWS Organizations
