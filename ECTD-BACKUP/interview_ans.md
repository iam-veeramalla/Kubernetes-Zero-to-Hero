For Kubernetes disaster recovery, we used two backup mechanisms: **etcd snapshots and Velero**.

We used **etcdctl** to take snapshots of the etcd database, which contains the Kubernetes cluster state, such as Kubernetes API resources and cluster configuration.

Along with that, we used **Velero** to back up our application workloads and Kubernetes resources.
For stateful applications, we also configured persistent volume protection so that the associated PV data could be recovered.

For the AWS setup, our workloads were using **Amazon EBS volumes through the EBS CSI driver**.

We install the EBS CSI driver and CSI snapshot components/CRDs. Then we create a VolumeSnapshotClass that references ebs.csi.aws.com. 
This tells the CSI snapshot mechanism to use the EBS CSI driver when creating snapshots of EBS-backed volumes.
We configured the required CSI snapshot components and a `VolumeSnapshotClass` for the EBS CSI driver.

Three main resources we will create by using the CSI snapshot components/CRDs

1. VolumeSnapshot: A namespaced user request or claim for a volume snapshot, similar to a PersistentVolumeClaim
2. VolumeSnapshotContent:  A cluster-scoped resource representing the actual snapshot provisioned on the physical storage backend, mapped one-to-one with a VolumeSnapshot
3. VolumeSnapshotClass: A cluster administrator resource that defines storage-provider-specific attributes and parameters used when dynamically creating a snapsho

        apiVersion: snapshot.storage.k8s.io/v1
        kind: VolumeSnapshotClass
        metadata:
          name: ebs-snapshot
          labels:
            velero.io/csi-volumesnapshot-class: "true"  -->The label tells Velero:"This VolumeSnapshotClass is available for Velero CSI backups."
        driver: ebs.csi.aws.com
        deletionPolicy: Delete
Velero was configured with the **AWS plugin/provider** and an **S3 bucket as the backup storage location**.

"We used IRSA to provide Velero with an IAM role. The IAM role had the required permissions to access the Velero S3 backup repository and, where applicable, 
the AWS APIs required for EBS snapshot operations. This allowed Velero to access AWS without storing long-lived access keys in the cluster."

For AWS authentication, we used **IRSA** to provide the required IAM permissions to the Kubernetes workloads. 
The IAM permissions were scoped according to the components' requirements, such as access to S3 and the necessary EBS snapshot APIs.

Option 1 — Let Velero create the ServiceAccount

      velero install \
        --provider aws \
        --plugins velero/velero-plugin-for-aws:<VERSION> \
        --bucket my-velero-backups \
        --backup-location-config region=us-east-1 \
        --service-account-name velero \
        --pod-annotations "eks.amazonaws.com/role-arn=arn:aws:iam::<ACCOUNT_ID>:role/VeleroBackupRole"

* While creating the IAM role itself we have to provide the trust entry in iam role for this velero service account.

  
option: 2 - this is the best approch

If you create the ServiceAccount yourself, make sure the namespace and ServiceAccount name match what Velero uses:
      
      velero install \
      --provider aws \
      --plugins velero/velero-plugin-for-aws:<VERSION> \
      --bucket my-velero-backups \
      --backup-location-config region=us-east-1 \
      --service-account-name velero


Policies for IAM roles:

for EBS CIS driver iam role: AmazonEBSCSIDriverPolicyV2

for Velero :

    s3:GetObject
    s3:PutObject
    s3:DeleteObject
    s3:ListBucket
    ec2:DescribeVolumes
    ec2:DescribeSnapshots
    ec2:CreateSnapshot
    ec2:DeleteSnapshot


Flow: 

    "Velero detects the PVC and uses Kubernetes CSI snapshot APIs. The appropriate VolumeSnapshotClass identifies the EBS CSI driver. The CSI snapshot controller                 and EBS CSI driver then handle the snapshot operation, resulting in an AWS EBS snapshot."

So, in summary:

**etcdctl → Kubernetes cluster state**

**Velero → Kubernetes resources/workloads + persistent volume protection**

**EBS CSI → EBS volume management and CSI snapshots**

**S3 → Velero backup repository**

This gave us separate recovery mechanisms for the Kubernetes control-plane state and the application/data layer.

