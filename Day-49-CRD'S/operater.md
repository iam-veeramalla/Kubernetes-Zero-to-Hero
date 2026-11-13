what is an Operator?
=====================

An Operator is software that extends Kubernetes to automate the deployment, management, and lifecycle of a particular application or system.

An Operator is a Kubernetes-native application that uses the controller/reconciliation pattern and application-specific knowledge
to automate the lifecycle and operational tasks of a particular application or system.

For example, imagine PostgreSQL.

A normal controller might know:

    "Keep 3 Pods running."

A PostgreSQL Operator can understand things like:

    How to deploy PostgreSQL
    How to configure it
    How to create storage
    How to create users
    How to perform backups
    How to handle upgrades
    How to detect failures
    How to restore a database
    How to manage replicas
    How to update the database safely

That's why we call it application-specific operational knowledge.


           Operator
                │
       ┌────────┴────────┐
       │                 │
    CRD/CR          Controller
       │                 │
       └────────┬────────┘
                ↓
           Reconciliation
                ↓
       Application resources


Why not just use a normal Controller?

This is the important question.

Suppose you have a custom resource:

Database

A basic custom controller could do:

    Database CR
        ↓
    Create StatefulSet
        ↓
    Create Service
        ↓
    Create PVC

But a real database has many operational requirements.

For example:

    Database
       ↓
    Install
       ↓
    Configure
       ↓
    Backup
       ↓
    Restore
       ↓
    Failover
       ↓
    Upgrade
       ↓
    Scale
       ↓
    Monitor
       ↓
    Recover

An Operator automates these day-2 operational tasks.

That's the major reason Operators are useful.

Operator = Kubernetes automation for the full lifecycle of an application.


Controller vs Operator
-----------------------

    | Controller                      | Operator                        |
    | ------------------------------- | ------------------------------- |
    | General control-loop concept    | Application-specific automation |
    | Watches resources               | Watches resources               |
    | Reconciles desired vs actual    | Reconciles desired vs actual    |
    | Can manage Kubernetes resources | Manages application lifecycle   |
    | May be simple                   | Usually more application-aware  |
    | Example: Deployment Controller  | Example: PostgreSQL Operator    |


Important:

      Controller
          =
      Observe + Reconcile
      
      Operator
          =
      Observe + Reconcile
              +
      Application knowledge
              +
      Lifecycle automation

An Operator is not a completely different mechanism from a Controller.

An Operator uses the controller pattern.

Real-world Operators
---------------------

cert-manager

    Automates TLS certificates.
    
    Certificate CR
          ↓
    cert-manager
          ↓
    Certificate
          ↓
    Secret
Prometheus Operator:

    Manages Prometheus monitoring resources.
    
    Prometheus CR
          ↓
    Prometheus Operator
          ↓
    Prometheus deployment/configuration
Strimzi: 

    Manages Apache Kafka on Kubernetes.
    
    Kafka CR
       ↓
    Strimzi Operator
       ↓
    Kafka cluster

CloudNativePG:

Automates PostgreSQL cluster deployment and lifecycle management.


We'll study each using the same approach: problem → CRD/CR → Operator → reconciliation → actual resources.

Operator 1 — cert-manager
-------------------------

Let's start with a real production use case.

The problem

Suppose you deploy an application on Kubernetes and expose it using HTTPS:

        https://app.example.com

For HTTPS to work correctly, you need a TLS certificate.

Without automation, an administrator may need to:

    Request a certificate from a certificate authority.
    
    Configure the certificate and private key.
    
    Store them in a Kubernetes Secret.
    
    Renew the certificate before it expires.
    
    Update the Secret when the certificate changes.

This creates repetitive operational work and risks certificate expiry.

cert-manager automates much of this lifecycle.

How cert-manager works:


Certificate CR:
    
    User requests a certificate

Example Certificate CR:

    apiVersion: cert-manager.io/v1
    kind: Certificate
    metadata:
      name: app-certificate
      namespace: production
    spec:
      secretName: app-tls  ----->Secret where certificate material is stored
      dnsNames:
        - app.example.com   -----> Domain names the certificate should cover
      issuerRef:              --------> Certificate issuer to use
        name: letsencrypt-prod
        kind: ClusterIssuer


The custom resource type

secretName: app-tls

	

Secret where certificate material is stored

dnsNames

	

Domain names the certificate should cover

issuerRef

	

Certificate issuer to use
cert-manager Controller:
    
    Watches resources and reconciles desired state
    
Certificate Issuance:
    
    Uses an Issuer or ClusterIssuer
    
Kubernetes TLS Secret:
    
    Stores the certificate and private key


Flow:

    The user applies the Certificate CR.
    
    The Kubernetes API Server validates and stores the resource.
    The cert-manager controller observes the resource and reconciles it.
    cert-manager works with the configured issuer to request and obtain the certificate.
    
    The certificate and private key are stored in the app-tls Secret.
    
    cert-manager tracks certificate status and renews it when needed.
        
Operator Architecture and Lifecycle
--------------------------------------

	1. User / DevOps Engineer
	
	Defines the desired database configuration
	
	2. Custom Resource (CR)
	
	For example, payment-db
	
	3. Kubernetes API Server
	
	Validates requests and stores resource state
	
	4. Operator Controller
	
	Watches resources and reconciles desired vs actual state
	
	5. Managed Kubernetes Resources
	
	StatefulSets or Pods, Services, PVCs, Secrets, etc.
	
	6. Status and Conditions
	
	Report the observed state of the application

One important detail: the API Server doesn't directly call the Operator every time a CR is created. The Operator watches relevant resources through Kubernetes API mechanisms and reconciles them.

The main components
--------------------

A. CRD — CustomResourceDefinition
	
	The CRD defines a new resource type that Kubernetes can recognize.
	
	Example:
	
	kind: Database
	
	The CRD specifies the resource's schema, supported versions, and whether it is namespaced or cluster-scoped.

B. CR — Custom Resource

	The CR is the actual object created by the user.
	
	apiVersion: database.example.com/v1
	kind: Database
	metadata:
	  name: payment-db
	spec:
	  engine: postgres
	  version: "16"
	  storage: 20Gi
	
	This declares the desired configuration. It doesn't create PostgreSQL by itself.

C. Operator controller

	The controller watches the CR and relevant managed resources.
	
	Its job is to:
	
	Read the desired state.
	
	Observe the current cluster state.
	
	Compare the two.
	
	Create, update, or remove resources as needed.
	
	Report the result in status and Conditions.

D. Managed resources

	Depending on the Operator design, it may manage resources such as:
	
	StatefulSets and Pods
	
	Services
	
	PersistentVolumeClaims
	
	Secrets and ConfigMaps
	
	Jobs used for backup or maintenance

E. Status and Conditions

	These help users and automation understand what is happening.
	
	For example:
	
	status:
	  conditions:
	    - type: Ready
	      status: "True"
	      reason: DatabaseReady
	      message: Database is accepting connections
	
	The exact fields depend on the Operator's API.



What happens during the Operator lifecycle?
--------------------------------------------

Imagine you're deploying a database for the first time.

Stage 1 — Installation

	The CRD and Operator are installed. Kubernetes learns the custom resource type, and the controller starts running.

Stage 2 — Resource creation

	The user creates a Database CR describing the desired database.

Stage 3 — Watch and reconcile

	The controller detects relevant changes and checks whether the desired resources exist.

Stage 4 — Provisioning

	The Operator creates or updates the required resources and waits for them to become ready.

Stage 5 — Ready

	Once the required health checks succeed, the Operator updates the resource's status and conditions.

Stage 6 — Ongoing management

	The Operator continues reconciling changes, handling failures, and performing supported lifecycle operations.

Remember: this is a conceptual lifecycle, not a strict one-time sequence. Reconciliation continues throughout the resource's life.

