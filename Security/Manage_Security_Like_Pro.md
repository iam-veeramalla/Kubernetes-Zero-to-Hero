# Kubernetes Security Journey for DevSecOps Engineers

As DevSecOps engineers, one of the primary resposibilities is to maintain security of your Kubernetes clusters and the containers.
Here are some of the mandatory things to consider.

1. [Secure your API server](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/blob/main/Security/Manage_Security_Like_Pro.md#secure-your-api-server)
2. [RBAC](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/blob/main/Security/Manage_Security_Like_Pro.md#rbac)
3. [Network Policies](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/blob/main/Security/Manage_Security_Like_Pro.md#network-policies)
4. [Encrypt data at rest](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/blob/main/Security/Manage_Security_Like_Pro.md#encrypt-data-at-rest)
5. [Secure Container Images](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/blob/main/Security/Manage_Security_Like_Pro.md#secure-container-images)
6. [Cluster Monitoring](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/blob/main/Security/Manage_Security_Like_Pro.md#cluster-monitoring)
7. [Upgrades](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/blob/main/Security/Manage_Security_Like_Pro.md#upgrades)

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
Use Role-Based Access Control to define who can access which resource in kubernetes. For example, not everyone should have access to kubernetes secrets.


## Network Policies
Use network policies to restrict traffic within the cluster and to/from external sources. 
Use firewalls and security groups to control traffic to and from the cluster.

## Encrypt data at rest
Use encryption to protect sensitive data stored in etcd and other components of the cluster.

To encrypt data at rest in Kubernetes, you can use the Kubernetes Encryption Provider feature, which encrypts sensitive data stored in etcd, the Kubernetes cluster's key-value store. The Encryption Provider uses a key management system to manage and store encryption keys.

Here are the general steps to enable Encryption Provider and encrypt data at rest in Kubernetes:

- [Enable the Encryption Provider feature by configuring the Kubernetes API server](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/blob/main/Security/Manage_Security_Like_Pro.md#enable-the-encryption-provider-feature-by-configuring-the-kubernetes-api-server)
- [Configure the key management system to store and manage encryption keys](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/blob/main/Security/Manage_Security_Like_Pro.md#configure-the-key-management-system-to-store-and-manage-encryption-keys)
- [Create a Kubernetes Secret object with the encryption key](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/blob/main/Security/Manage_Security_Like_Pro.md#create-a-kubernetes-secret-object-with-the-encryption-key)
- [Configure Kubernetes resources to use the Encryption Provider](https://github.com/iam-veeramalla/Kubernetes-Zero-to-Hero/blob/main/Security/Manage_Security_Like_Pro.md#configure-kubernetes-resources-to-use-the-encryption-provider)

Let's dive into these steps in more detail:

#### Enable the Encryption Provider feature by configuring the Kubernetes API server.

You can enable the Encryption Provider feature by adding the --encryption-provider-config option to the Kubernetes API server command-line arguments or to the API server manifest file. This option points to a configuration file that specifies the encryption provider and its settings.

Here is an example of a simple encryption provider configuration file:

```
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources: ["secrets"]
    providers:
      - identity: {}
```

In this example, we are enabling the Encryption Provider for Secrets resources using the "identity" provider, which uses a default encryption algorithm and key size.

#### Configure the key management system to store and manage encryption keys.

The Encryption Provider requires a key management system to store and manage encryption keys. You can use a cloud-based key management system like Google Cloud KMS or Amazon Web Services KMS, or a self-hosted key management system like HashiCorp Vault.

You must configure the key management system to generate a key and give the Kubernetes API server access to the key. The specific steps to do this depend on the key management system you are using.

#### Create a Kubernetes Secret object with the encryption key.

Once you have a key from your key management system, you can create a Kubernetes Secret object that stores the key. You can create the Secret object using kubectl:

`kubectl create secret generic encryption-key --from-literal=encryption-key=<base64-encoded-key>`

In this example, we are creating a Secret object called "encryption-key" and storing the key as a base64-encoded literal.

#### Configure Kubernetes resources to use the Encryption Provider.

To use the Encryption Provider to encrypt data at rest, you need to configure Kubernetes resources to use the feature. You can do this by setting the encryptionConfig field in the Kubernetes API server manifest file or by setting the metadata.annotations field in the Kubernetes resource definition.

Here is an example of a Kubernetes Secret definition that uses the Encryption Provider:

```
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  annotations:
    encryptionConfig: secrets
type: Opaque
data:
  username: <base64-encoded-username>
  password: <base64-encoded-password>
```

In this example, we are defining a Secret object called "my-secret" that contains sensitive data. We are setting the metadata.annotations field to specify that the Encryption Provider should be used to encrypt the data. The data field contains the sensitive data, which is base64-encoded.

By following these steps, you can enable and configure the Kubernetes Encryption Provider feature to encrypt data at rest in your Kubernetes cluster.

## Secure Container Images
Use container images from trusted sources and scan them for vulnerabilities before deployment.

To scan images for vulnerabilities you can use simple commands like
`docker scan --severity high <docker-image-location>`

## Cluster Monitoring
Use tools like Kubernetes Audit Logs and security monitoring solutions to detect and respond to security threats in real-time.

![Screenshot 2023-03-05 at 8 16 52 PM](https://user-images.githubusercontent.com/43399466/222967569-8b05f3e3-ead6-4943-b6ce-3db3af242a5a.png)


## Upgrades
Keep the Kubernetes cluster and its components up to date with the latest security patches and updates.
