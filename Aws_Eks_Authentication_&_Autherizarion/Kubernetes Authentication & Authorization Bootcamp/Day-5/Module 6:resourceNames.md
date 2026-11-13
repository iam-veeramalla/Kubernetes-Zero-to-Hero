What problem does resourceNames solve?
=============================================
Normally, if a user has permission to read ConfigMaps, they can read every ConfigMap in the namespace.

Example:

    kubectl get configmaps

RBAC:

    resources:
    - configmaps
    
    verbs:
    - get

Result:

      config1
      config2
      config3
      database-config
      secret-config

Sometimes you don't want that.

You only want the user to access one specific resource.
--------------------------------------------------------
This is where resourceNames is used.

Example

Suppose your application has:

ConfigMaps

    database-config
    application-config
    redis-config
    nginx-config

Your application should only read:

    database-config

RBAC:

      apiVersion: rbac.authorization.k8s.io/v1
      kind: Role
      
      metadata:
        name: app-reader
        namespace: production
      
      rules:
      - apiGroups: [""]
        resources:
        - configmaps
      
        resourceNames:
        - database-config
      
        verbs:
        - get

Now:

kubectl get configmap database-config

    ✅ Allowed

But:

kubectl get configmap redis-config

    ❌ Forbidden

Where is it used?

Production examples:

One application should read only its own ConfigMap.
One Job should update only one Secret.
Backup tool can access only one PVC.
Monitoring tool can read only one Lease object.

This is an important least-privilege feature.
