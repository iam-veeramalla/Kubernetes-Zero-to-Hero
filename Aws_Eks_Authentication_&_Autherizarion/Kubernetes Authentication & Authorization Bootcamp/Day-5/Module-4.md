Module 4: ClusterRole Used Inside RoleBinding
==============================================
Many engineers think:

ClusterRole

↓

Must use

↓

ClusterRoleBinding

This is false.

A ClusterRole can be reused in a namespace through a RoleBinding.

Example:

ClusterRole

view

Bind it:

RoleBinding

Namespace: dev

↓

ClusterRole

view

Result:

The user gets view permissions only inside the dev namespace, even though the source is a ClusterRole.

This avoids duplicating common permission sets across namespaces.
