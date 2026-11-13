Module 1: How RBAC Evaluation Actually Works
==============================================

Let's first understand what happens when a request reaches the API server.
Suppose a developer runs:

kubectl delete pod nginx

The flow looks like this:

                        kubectl delete pod nginx
                                │
                                ▼
                        Authentication
                                │
                                ▼
                        User Identity = dev-user
                                │
                                ▼
                        Authorization (RBAC)
                        
                        Find all RoleBindings
                                │
                                ▼
                        Find all ClusterRoleBindings
                                │
                                ▼
                        Collect all Roles & ClusterRoles
                                │
                                ▼
                        Merge all permissions
                                │
                                ▼
                        Is delete on pods allowed?

YES → Execute
NO  → 403 Forbidden

*Important Rule*
------------------

RBAC does not stop at the first RoleBinding.

It collects permissions from every applicable RoleBinding and ClusterRoleBinding, then combines them.

Think of it as a union of permissions.

Example:

Role A

get pods

-----------------

Role B

create deployment

-----------------

Effective Permissions

get pods
create deployment

Permissions are added together.
