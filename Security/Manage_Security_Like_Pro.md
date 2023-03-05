# Kubernetes Security Journey for DevSecOps Engineers

As DevSecOps engineers, one of the primary resposibilities is to maintain security of your Kubernetes clusters and the containers.
Here are some of the mandatory things to consider.

1. [Secure your API server](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/edit/main/Security/Manage_Security_Like_Pro.md#secure-your-api-server)
2. [RBAC](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/edit/main/Security/Manage_Security_Like_Pro.md#rbac)
3. [Network Policies](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/edit/main/Security/Manage_Security_Like_Pro.md#network-policies)
4. [Encrypt data at rest](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/edit/main/Security/Manage_Security_Like_Pro.md#encrypt-data-at-rest)
5. [Secure Container Images](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/edit/main/Security/Manage_Security_Like_Pro.md#secure-container-images)
6. [Cluster Monitoring](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/edit/main/Security/Manage_Security_Like_Pro.md#cluster-monitoring)
7. [Upgrades](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/edit/main/Security/Manage_Security_Like_Pro.md#upgrades)

## Secure your API server
The Kubernetes API server is a critical component of the cluster and should be secured with strong authentication and authorization mechanisms. 
Use TLS certificates for all communications with the API server.

1. Enable TLS encryption
2. Use strong authentication
3. Restrict access
4. Monitor and audit
5. Keep the API server up to date

#### Enable TLS encryption

In your Kubernetes API server configuration file (kube-apiserver.yaml), add the following lines to enable TLS encryption:

```
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
spec:
  containers:
  - name: kube-apiserver
    image: k8s.gcr.io/kube-apiserver:v1.21.0
    command:
    - kube-apiserver
    - --tls-cert-file=/path/to/server.crt
    - --tls-private-key-file=/path/to/server.key
    - --client-ca-file=/path/to/ca.crt
```

In this example, we are using server.crt and server.key files for the server certificate and private key respectively, and ca.crt for the client certificate authority.

#### Use strong authentication

In the same kube-apiserver.yaml file, configure the authentication method you prefer, such as client certificates or bearer tokens. Here is an example of how to use client certificates.

```
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
spec:
  containers:
  - name: kube-apiserver
    image: k8s.gcr.io/kube-apiserver:v1.21.0
    command:
    - kube-apiserver
    - --tls-cert-file=/path/to/server.crt
    - --tls-private-key-file=/path/to/server.key
    - --client-ca-file=/path/to/ca.crt
    - --authentication-mode=x509
    - --requestheader-client-ca-file=/path/to/ca.crt
    - --requestheader-allowed-names=""
    - --requestheader-extra-headers-prefix="X-Remote-Extra-"
```

In this example, we are using x509 authentication and configuring the request headers to include X-Remote-Extra- headers. This allows you to pass additional information about the client, such as the user's email or group membership.

#### Restrict access

Configure RBAC to restrict access to the Kubernetes API server based on roles and permissions. Here is an example of how to configure RBAC.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: api-server-reader
rules:
- apiGroups: [""]
  resources: ["pods", "namespaces"]
  verbs: ["get", "watch", "list"]

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: api-server-reader-binding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: api-server-reader
subjects:
- kind: User
  name: alice
```

In this example, we are creating a ClusterRole called api-server-reader that allows read-only access to pods and namespaces, and a RoleBinding that assigns this role to the user alice in the default namespace.

#### Monitor and audit

Use tools like Kubernetes Audit to monitor and audit the Kubernetes API server. Here is an example of how to configure Kubernetes Audit.

```
apiVersion: audit.k8s.io/v1beta1
kind: Policy
rules:
- level: Metadata
```

#### Keep the API server up to date

Make sure to keep the Kubernetes API server up to date with the latest security patches and updates by regularly checking for updates and applying them as needed.

By following these steps, you can enhance the security of the Kubernetes API.

## RBAC
Use Role-Based Access Control to define who can access the Kubernetes API and what actions they are allowed to perform. 
Use strong authentication methods like multi-factor authentication and enforce password policies.

## Network Policies
Use network policies to restrict traffic within the cluster and to/from external sources. 
Use firewalls and security groups to control traffic to and from the cluster.

## Encrypt data at rest
Use encryption to protect sensitive data stored in etcd and other components of the cluster.

## Secure Container Images
Use container images from trusted sources and scan them for vulnerabilities before deployment.

## Cluster Monitoring
Use tools like Kubernetes Audit Logs and security monitoring solutions to detect and respond to security threats in real-time.

## Upgrades
Keep the Kubernetes cluster and its components up to date with the latest security patches and updates.
