# Velero — Kubernetes Disaster Recovery

## 1. Why Disaster Recovery Matters

* Hardware failures, human errors, and security breaches can destroy cluster data.
* Minimize downtime and recovery time objectives (RTO).
* Enable cluster migration and environment replication.

---

# 2. What is Velero?

Velero is an open-source tool for backing up and restoring Kubernetes cluster resources and persistent volumes.

### Key Features

* Cloud-agnostic solution
* Backup entire namespaces
* Selective resource backup
* Secure object storage

---

# 3. Velero Architecture

### Server Component

Runs as a deployment in the cluster and performs backup/restore operations.

### CLI Client

Command-line tool to trigger backups, restores, and manage schedules.

### Object Storage

External storage such as:

* S3
* Azure Blob
* GCS

Object storage is used for backup data persistence.

---

# 4. AWS Prerequisites

For running Velero with **AWS/EKS**, make sure the following are available.

### 1. AWS Account

You need an AWS account with permission to create and manage the required resources.

### 2. S3 Bucket

Create an S3 bucket to store Velero backup data.

Example:

```text
S3 Bucket
   ↓
velero-backups
   ↓
Kubernetes backup data
```

### 3. IAM Permissions

Velero needs AWS permissions to access the S3 bucket and, when using volume snapshots, the required EBS/EC2 snapshot APIs.

Typical permissions include:

* S3 bucket access
* S3 object read/write/delete
* EBS snapshot permissions
* EC2-related permissions required by the AWS Velero plugin

For production, use an **IAM role with least-privilege permissions** rather than broad administrator permissions.

### 4. AWS Credentials / IAM Role

Velero needs AWS credentials to access AWS resources.

For EKS, an IAM role associated with the Velero service account is a common approach.

```text
Velero Pod
    ↓
IAM Role
    ↓
AWS APIs
    ↓
S3 / EBS
```

### 5. AWS Region

Know the AWS region where your cluster and backup resources are located.

Example:

```text
us-east-1
```

The S3 bucket and other AWS resources should be configured appropriately for your backup strategy.

### 6. EBS CSI Driver — If Using EBS Volumes

If your Kubernetes workloads use AWS EBS volumes and you want to protect their persistent data using CSI snapshots, make sure the **Amazon EBS CSI driver** is installed and configured.

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
EBS Volume
 ↓
EBS CSI Driver
 ↓
Snapshot
```

### 7. CSI Snapshot Components

For CSI-based volume snapshots, the Kubernetes cluster needs the appropriate CSI snapshot components and the storage driver's snapshot support.

### 8. Velero AWS Plugin

Velero needs the AWS provider/plugin to interact with AWS services such as S3 and EBS.

---

# 5. Installing Velero

### Download Velero CLI

```bash
wget https://github.com/vmware-tanzu/velero/releases/...
```

### Install Velero in the Cluster

```bash
velero install \
  --provider aws \
  --bucket velero-backups \
  --secret-file ./credentials-velero \
  --backup-location-config region=us-east-1
```

---

# 6. Creating Backups

### Backup Entire Cluster

```bash
velero backup create full-cluster-backup
```

### Backup Specific Namespace

```bash
velero backup create app-backup \
  --include-namespaces production
```

### Backup Using Label Selector

```bash
velero backup create label-backup \
  --selector app=nginx
```

### Check Backup Status

```bash
velero backup describe full-cluster-backup
```

---

# 7. Scheduled Backups

Automate backups using cron expressions for regular intervals.

### Daily Backup at 2 AM

```bash
velero schedule create daily-backup \
  --schedule="0 2 * * *"
```

### Weekly Backup with 30-Day Retention

```bash
velero schedule create weekly-backup \
  --schedule="0 0 * * 0" \
  --ttl 720h0m0s
```

---

# 8. Restoring from Backups

### List Available Backups

```bash
velero backup get
```

### Restore Entire Backup

```bash
velero restore create \
  --from-backup full-cluster-backup
```

### Restore Specific Namespace

```bash
velero restore create \
  --from-backup app-backup \
  --include-namespaces production
```

### Check Restore Status

```bash
velero restore describe restore-name
```

---

# 9. Backup Best Practices

### 3-2-1 Backup Rule

Implement the 3-2-1 rule:

* 3 copies of data
* 2 different media
* 1 offsite copy

### Regularly Test Restores

Regularly test restore procedures to ensure backup integrity.

### Encrypt Backups

Encrypt backups and securely manage access to storage credentials.

### Define Retention Policies

Define retention policies based on compliance requirements.

---

# 10. Key Takeaways

* Velero provides comprehensive backup and restore capabilities for Kubernetes clusters.
* AWS S3 can be used to store Velero backup data.
* IAM permissions are required for Velero to access AWS resources.
* For EBS persistent volumes, configure the EBS CSI driver and an appropriate volume-backup/snapshot mechanism.
* Automated scheduled backups ensure continuous data protection.
* Regular testing and adherence to best practices are critical for successful disaster recovery.
