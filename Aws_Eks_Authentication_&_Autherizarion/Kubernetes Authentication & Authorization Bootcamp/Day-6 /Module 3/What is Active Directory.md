What is Active Directory ?

Microsoft created

Active Directory (AD)

It is LDAP plus many enterprise features.

It stores

Users
Groups
Computers
Passwords
Organizational Units (OUs)
Policies

Example:
Company

|
+-- DevOps

      Rahul

      John

|
+-- Developers

      Mike

      David

|
+-- QA

      Priya


Groups become very useful for Kubernetes RBAC.

Example:

AD Group

DevOps-Team

↓

ClusterRoleBinding

↓

cluster-admin

Every DevOps engineer gets cluster access automatically.
