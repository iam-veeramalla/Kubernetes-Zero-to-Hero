Module 7: Non-Resource URLs
=====================
Not everything in Kubernetes is a resource like Pods or Deployments.

Some API endpoints are URLs.

Examples:

      /healthz
      /version
      /metrics
      /livez
      /readyz
      /openapi

To control access to these endpoints, use nonResourceURLs.
--------------------------------------------------------------
Example:

    rules:
    - nonResourceURLs:
      - /healthz
      verbs:
      - get

This allows:

kubectl get --raw /healthz

but not access to other URLs.

Production Use Case
------------------
Monitoring systems like Prometheus may need to scrape:

/metrics

You can grant only:

    rules:
    - nonResourceURLs:
      - /metrics
      verbs:
      - get

instead of broader API access.
