Module 3: Multiple RoleBindings
===============================
Suppose you have:

Namespace: dev

        Role:
        Pod Reader
        
        ↓
        
        RoleBinding 1
        
        ↓
        
        developer
        
        Another Role:
        
        Deployment Admin
        
        ↓
        
        RoleBinding 2
        
        ↓
        
        developer
        
        Effective permissions:
        
        Read Pods
        
        +
        
        Manage Deployments
        
        RBAC combines them.
