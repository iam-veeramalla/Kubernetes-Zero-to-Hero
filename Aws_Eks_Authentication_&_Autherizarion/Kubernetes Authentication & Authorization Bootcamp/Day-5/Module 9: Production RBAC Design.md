Module 9: Production RBAC Design

Imagine your organization has:
---------------------------------

      Platform Team
      DevOps Team
      Developers
      QA
      Security
      CI/CD Systems
      Monitoring

A common RBAC design looks like this:
--------------------------------------
| Identity      | Permissions                                                 |
| ------------- | ----------------------------------------------------------- |
| Platform Team | `cluster-admin`                                             |
| DevOps Team   | Cluster-wide operational permissions (custom `ClusterRole`) |
| Developers    | Only their application namespaces                           |
| QA            | Read pods, logs, exec in test namespaces                    |
| Security      | Read RBAC, NetworkPolicies, Secrets (audit only)            |
| Jenkins       | Deploy to application namespaces                            |
| Argo CD       | Sync applications                                           |
| Prometheus    | Read cluster resources required for monitoring              |

Notice that almost nobody except the platform administrators receives cluster-admin.
