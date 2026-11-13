Argo-CD
--------

What is Git-Ops ?
------------------

* Gitops uses git as a single source of truth to deliver applications and infra.
* we can track the changes via Git what are modified in the app or infra.
* It is very useful when we are working with n no of clusters.

Gitops-Principles :
-------------------

* Declarative: a system managed by Gitops must have its desired state expressed declaratively.

* versioned and immutable: desired state is stored in a way that enforced immutability, versioning and retains a complete version history.

       “We save every change, never overwrite anything, and we can go back to any previous version.”

* pulled automatically: software agents automatically pull the desired state declaration from the source.

        Argo CD will support pull and push mechanism.

* continuously reconciled: software agent continuously observer actual system state and attempt to apply the desired state.

        if some one try to make changes in K8s cluster resources gitops controller Argo CD  will not allow them to make changes directly.
        
        Git ops controller have read access to all the resources in the cluster and it will keep catch.


Is GITOPS for K8S only ?
------------------------

* By principal the ans is Nooooooo

* Put the popular gitops tool like ARGO CD and flux target k8s.

Advantages of GITOPS:
---------------------

- security
- versioning - Argo CD will support s3 for versioning not only a git 
- auto upgrade 
- auto healing of any unwanted changes
- continuous reconciliation


