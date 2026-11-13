Module 10: Common RBAC Mistakes
================================
Mistake 1
----------
Giving everyone:

cluster-admin

❌ Very dangerous.

Mistake 2
----------
Using the default ServiceAccount everywhere.

❌ All applications inherit the same permissions.

Mistake 3
-----------
Using *

resources:
- "*"

verbs:
- "*"

❌ Violates the principle of least privilege.

Mistake 4
---------
Creating many duplicate Roles.

Instead, reuse ClusterRoles where appropriate and bind them with RoleBindings or ClusterRoleBindings.
