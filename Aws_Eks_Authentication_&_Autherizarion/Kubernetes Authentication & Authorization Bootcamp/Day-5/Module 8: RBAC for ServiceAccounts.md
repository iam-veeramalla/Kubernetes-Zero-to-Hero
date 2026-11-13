Module 8: RBAC for ServiceAccounts
====================================
So far we've discussed human users.

Applications also need permissions.

Applications use ServiceAccounts.

Example:

Jenkins
Argo CD
Prometheus
External Secrets
cert-manager

Each application gets its own ServiceAccount.

Example:

apiVersion: v1
kind: ServiceAccount

metadata:
  name: jenkins
  namespace: cicd

Now create a Role:
---------------------
kind: Role

rules:

- apiGroups: [""]

  resources:
  - pods

  verbs:
  - get
  - list
  - watch

Bind it:
----------
kind: RoleBinding

subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: cicd

Now only the jenkins ServiceAccount gets these permissions.

Why not use the default ServiceAccount?
------------------------------------------------------------
Many beginners deploy applications using:

default

This is a bad practice because multiple applications end up sharing the same identity and permissions.

Production rule:

One application = One ServiceAccount.
Give each ServiceAccount only the permissions it needs.
