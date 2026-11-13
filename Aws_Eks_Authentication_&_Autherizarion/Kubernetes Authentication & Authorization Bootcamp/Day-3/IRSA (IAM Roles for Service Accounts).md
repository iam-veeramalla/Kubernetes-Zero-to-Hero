Topic 3: IRSA (IAM Roles for Service Accounts)
----------------------------------------------


Learning Objectives
--------------------
By the end of this topic, you'll understand:

* Why IRSA was introduced
* Problems before IRSA
* How IRSA works internally
* OIDC Provider
* AWS STS (Security Token Service)
* Complete authentication flow
* Production implementation
* Interview questions

What Problem Does IRSA Solve?
------------------------------
Imagine you have an application running inside an EKS Pod.

      +----------------------+
      |   Inventory App      |
      |                      |
      | Needs to access S3   |
      +----------------------+

The application wants to upload files to S3.

Question:

How does the application authenticate with AWS?

Before IRSA
------------
Before IRSA, many companies used AWS Access Keys.

      AWS_ACCESS_KEY_ID
      
      AWS_SECRET_ACCESS_KEY

stored as

Kubernetes Secret

Example

      apiVersion: v1
      kind: Secret
      
      data:
        AWS_ACCESS_KEY_ID: xxxx
        AWS_SECRET_ACCESS_KEY: xxxx
   -----------------------------------

      Application
      
      ↓
      
      Reads Secret
      
      ↓
      
      Uses AWS Access Key
      
      ↓

    Calls S3

Problems
--------
Problem 1

    Long-lived credentials

    Access Key
    
    ↓
    
    Never changes

If leaked,

attacker gets AWS access.

Secrets stored inside Kubernetes

      etcd
      
      ↓
      
      AWS Credentials

Not ideal.

Problem 3

Credential rotation

Someone has to manually rotate keys.

Huge operational burden.

Problem 4

All Pods often shared the same credentials.

      Pod A
      
      ↓
      
      AWS Keys
      
      ↓
      
      S3
      Pod B
      
      ↓
      
      Same AWS Keys
      
      ↓
      
      S3

Not secure.

AWS Solution → IRSA
-----------------------
Instead of storing AWS credentials,

AWS says:

"Use your Kubernetes ServiceAccount identity."

So now

      Pod
      
      ↓
      
      ServiceAccount
      
      ↓
      
      Bound JWT Token
      
      ↓
      
      AWS validates it
      
      ↓
      
      Returns Temporary AWS Credentials

What does IRSA stand for?
--------------------------

IAM Roles for Service Accounts

Meaning:

Attach an AWS IAM Role to a Kubernetes ServiceAccount.

Instead of

    Pod
    
    ↓
    
    AWS Keys

We use

    Pod
    
    ↓
    
    ServiceAccount
    
    ↓
    
    IAM Role

Real Flow
----------
Suppose

    Pod
    
    ↓
    
    ServiceAccount
    
    ↓
    
    app-sa

And

    IAM Role
    
    ↓
    
    S3ReadOnlyRole

is attached to

app-sa

Now the Pod wants S3.

Instead of using AWS keys,

it sends

Bound Service Account Token

to AWS.

Wait...
---------
AWS doesn't know Kubernetes.

How can AWS trust this JWT?

Excellent question.

This is where

OIDC Provider

comes into the picture.

What is OIDC?
--------------
    OIDC
    
    =
    
    OpenID Connect

It is an identity protocol.

Your EKS cluster publishes an OIDC endpoint.

Example

    https://oidc.eks.ap-south-1.amazonaws.com/id/ABC123XY

Think of it as:
    
    EKS
    
    ↓
    
    Identity Provider

AWS IAM is configured to trust this identity provider.


Trust Relationship
------------------
Imagine

    Kubernetes
    
    ↓
    
    "I created this JWT."

AWS asks

Can I trust you?

OIDC answers

Yes.

I'm the official identity provider for this EKS cluster.

Now AWS trusts the JWT.

Complete Authentication Flow

Let's understand every step.

Step 1

    Pod starts.
    
    Pod
    
    ↓
    
    Uses ServiceAccount app-sa
Step 2

Kubelet requests a Bound Service Account Token.

    Kubelet
    
    ↓
    
    TokenRequest API
    
    ↓
    
    API Server
    
    ↓
    
    JWT Token
Step 3

      JWT mounted inside Pod.
      
      /var/run/secrets/kubernetes.io/serviceaccount/token
Step 4

    Application wants S3.

Instead of

    AWS Access Key

it sends

    JWT Token

to

    AWS STS

What is AWS STS?
----------------
    STS
    
    =
    
    Security Token Service.

Its job is

    Validate Identity
    
    ↓
    
    Issue Temporary AWS Credentials

Step 5

    STS verifies
    
    JWT
    
    using
    
    OIDC Provider
    
    If valid
    
    ↓
    
    STS generates
    
    Temporary Access Key
    
    Temporary Secret Key
    
    Session Token

Step 6

    Application receives
    
    Temporary Credentials
    
    and calls
    
    Amazon S3
    
Entire Flow


      Application
      
              │
              ▼
      Bound JWT Token
      
              │
              ▼
      AWS STS
      
              │
              ▼
      Validate JWT
      
              │
              ▼
      OIDC Provider
      
              │
              ▼
      Yes, Valid
      
              │
              ▼
      Temporary AWS Credentials
      
              │
              ▼
              S3
      
Why is this Secure?

Because

No

AWS_ACCESS_KEY

AWS_SECRET_KEY

inside Pods.

Instead

Short-lived JWT

↓

Temporary AWS Credentials

↓

Automatic Expiration
Production Example

Suppose

Prometheus

needs CloudWatch.

Prometheus

↓

ServiceAccount

↓

prometheus-sa

↓

IAM Role

↓

CloudWatchReadOnly

↓

CloudWatch

No AWS keys.

Another example

ExternalDNS

needs Route53.

ExternalDNS

↓

ServiceAccount

↓

externaldns-sa

↓

IAM Role

↓

Route53Access
--------------

IRSA Setup (High-Level)

Step 1

Create OIDC Provider

    eksctl utils associate-iam-oidc-provider \
    --cluster demo \
    --approve

Step 2

Create IAM Policy

Example

    s3:GetObject
    
    s3:PutObject

Step 3

    Create IAM Role
    
    Attach Policy.

Step 4

Trust Policy

Very important.

Trust only

    system:serviceaccount:default:app-sa

This ensures only that specific ServiceAccount can assume the IAM Role.

Step 5

Annotate ServiceAccount

    apiVersion: v1
    kind: ServiceAccount
    
    metadata:
    
      name: app-sa
    
      annotations:

      eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/S3ReadOnlyRole

This annotation tells EKS:

"Whenever a Pod uses this ServiceAccount, allow it to assume this IAM Role."

Step 6

Create Pod
    
    spec:
    
      serviceAccountName: app-sa

Done.

The application now receives temporary AWS credentials automatically through the AWS SDK.

Old vs IRSA

      | Old Method          | IRSA                      |
      | ------------------- | ------------------------- |
      | AWS Access Keys     | IAM Role                  |
      | Long-lived          | Temporary                 |
      | Kubernetes Secret   | Bound JWT                 |
      | Manual Rotation     | Automatic                 |
      | High Risk           | Least Privilege           |
      | Credentials in Pods | No static AWS credentials |

Real Company Scenario
------------------------
Suppose your company has:

Jenkins
ExternalDNS
AWS Load Balancer Controller
Cluster Autoscaler

Each application gets its own ServiceAccount and its own IAM Role.

Jenkins
        │
        ▼
jenkins-sa
        │
        ▼
IAM Role
        │
        ▼
ECR + S3


ExternalDNS
        │
        ▼
externaldns-sa
        │
        ▼
IAM Role
        │
        ▼
Route53


AWS Load Balancer Controller
        │
        ▼
aws-lbc-sa
        │
        ▼
IAM Role
        │
        ▼
ELB Permissions

Each application has only the permissions it needs.

This follows the principle of least privilege.

Interview Questions
--------------------
1. What is IRSA?

IRSA (IAM Roles for Service Accounts) is an EKS feature that allows a Kubernetes ServiceAccount to assume an AWS IAM Role using a Bound Service Account Token and OIDC, eliminating the need to store static AWS credentials in Pods.

2. Why is IRSA more secure than storing AWS Access Keys?

Because it uses:

Short-lived Kubernetes JWTs
Temporary AWS credentials from STS
No static AWS access keys
Automatic credential rotation
Least-privilege IAM roles

3. What is the role of OIDC in IRSA?

The EKS cluster exposes an OIDC identity provider. AWS IAM trusts this provider and uses it to verify the Bound Service Account Token presented by the Pod before allowing the IAM Role to be assumed.

4. What is the role of AWS STS?

AWS STS validates the Kubernetes JWT and, if it satisfies the IAM Role's trust policy, issues temporary AWS credentials (Access Key, Secret Access Key, and Session Token) that the application uses to access AWS services.

🎯 Interview Challenge
-------------------------

Let's see if you've connected all the concepts.

Suppose a Pod using the ServiceAccount app-sa needs to upload a file to Amazon S3.

Without mentioning AWS Access Keys, explain the complete authentication flow from:

Pod starts → File uploaded to S3

Include these components in your explanation:

Kubelet
TokenRequest API
Bound Service Account Token
OIDC Provider
AWS STS
IAM Role
Amazon S3

If you can explain that flow correctly, you'll have mastered one of the most valuable EKS security concepts for DevOps interviews.
