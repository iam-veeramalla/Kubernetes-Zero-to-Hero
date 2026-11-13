Module 5: Aggregated ClusterRoles
==================================
Imagine your cluster installs many Custom Resource Definitions (CRDs).

Example:

      Prometheus
      
      Istio
      
      Argo CD
      
      Cert Manager

Each introduces new API resources.

Instead of editing the built-in Clutser Roles:

     view, edit, or admin 
    
For example, the built-in view role allows a user to read normal Kubernetes resources:


    Pods
    Services
    Deployments
    ConfigMaps
    ...

Now you install Prometheus Operator.

It introduces a new CRD:

    ServiceMonitor

The built-in view role doesn't automatically know that users should be allowed to read ServiceMonitor.

So you could manually edit the view ClusterRole and add:
Example labels:


      - apiGroups:
          - monitoring.coreos.com
        resources:
          - servicemonitors
        verbs:
          - get
          - list
          - watch

But manually modifying built-in roles is not a good approach.

That's where Aggregated ClusterRoles come in.

What is an Aggregated ClusterRole?
--------------------------------------
Think of it as:

"Add these RBAC permissions to an existing built-in ClusterRole."

You create a separate ClusterRole containing permissions for your custom resources.

For example:


    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: prometheus-view
      labels:
        rbac.authorization.k8s.io/aggregate-to-view: "true"
    rules:
    - apiGroups:
        - monitoring.coreos.com
      resources:
        - servicemonitors
      verbs:
        - get
        - list
        - watch
    


Notice this label:

    rbac.authorization.k8s.io/aggregate-to-view: "true"

This tells Kubernetes:

      "Take the RBAC rules from this ClusterRole and add them to the built-in view ClusterRole."

What happens internally?
-------------------------
Initially:

    Built-in view ClusterRole
            |
            +-- Pods: get/list/watch
            +-- Services: get/list/watch
            +-- Deployments: get/list/watch

You create:

    prometheus-view ClusterRole
            |
            +-- ServiceMonitor: get/list/watch

with:

    aggregate-to-view: "true"

Kubernetes' ClusterRole aggregation controller notices this label.

It effectively builds:

                       view
                        |
              +---------+----------+
              |                    |
         Normal K8s rules     Aggregated rules
              |                    |
            Pods              ServiceMonitor
            Services
            Deployments

So the effective permissions of view become:

      view
       |
       +-- Pods
       +-- Services
       +-- Deployments
       +-- ConfigMaps
       +-- ServiceMonitors   <-- added by aggregation


There are three common aggregation labels
------------------------------------------
View
rbac.authorization.k8s.io/aggregate-to-view: "true"

Adds permissions to:

view

Usually used for read-only access.

Edit
rbac.authorization.k8s.io/aggregate-to-edit: "true"

Adds permissions to:

edit

Usually used for resources that users need to modify.

Admin
rbac.authorization.k8s.io/aggregate-to-admin: "true"

Adds permissions to:

admin

Usually gives broader management permissions.

Important point: aggregation doesn't give access by itself
----------------------------------------------------------
This is a very important concept.

Suppose you create:


    kind: ClusterRole
    metadata:
      name: prometheus-view
      labels:
        rbac.authorization.k8s.io/aggregate-to-view: "true"
    rules:
    - apiGroups:
        - monitoring.coreos.com
      resources:
        - servicemonitors
      verbs:
        - get
        - list
        - watch


This doesn't mean every user can now read ServiceMonitors.

Instead:


    prometheus-view
           ↓
    aggregated into
           ↓
    view
           ↓
    users who have "view"
           ↓
    can now read ServiceMonitors


So aggregation modifies the permissions contained in the target ClusterRole.

The user still needs a RoleBinding or ClusterRoleBinding to the view role.




