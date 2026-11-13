How we can took backup by using velero to s3
============================================


* Before taking backup we have to install velero
* Velero has two major components to install
  1. Client--> which we have to install on laptop
  2. server --> which we have to install on cluster (master node)
 
1. Velero Client installtion:
   -------------------------

   * Go to velero official page velero for aws --> https://velero.io/docs/v1.0.0/aws-config/
   * Go to *official releases* in the page and scrool down and get the wget velero-v1.18.2-linux-amd64.tar.gz --tar.gz link.
   * untar it will tar -zxvf velero-v1.18.2-linux-amd64.tar.gz
   * we will get velero directory
   * cd velero directory and cp the velero folder to /usr/local/bin folder.

2. create s3 bucket as instrcted in the offficial docs.
3. create iam user, create policy and attch the policy to user.
4. Create an access key for the user
5. install the velero step in that cmd we have pass one flag called **--plugins velero/velero-plugin-for-aws:*version we have to metion**


To take a back up by using a velero:
-------------------------------------

CMD to take namespace backup in k8s:

         velero backup create <backup_name> --include-namespaces <name_Sapce_name>
to see the list of backups:

          velero get backups
to restore a backup:

          velero restore create --from-backup <backup_name>


IN EKS CLUSTER SETUP:
----------------------

* In EKS cluster add this add on called EBS CSI drivers:

      aws eks describe-addon \
        --cluster-name <cluster-name> \
        --addon-name aws-ebs-csi-driver

2. create the IAM role. AWS currently recommends using EKS Pod Identity for add-on IAM permissions, although IRSA is still supported.

         eksctl create iamserviceaccount \
        --name ebs-csi-controller-sa \
        --namespace kube-system \
        --cluster <cluster-name> \
        --role-name AmazonEKS_EBS_CSI_DriverRole \
        --role-only \
        --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicyV2 \
        --approve

3. Install CSI Snapshot Controller.
   EBS CSI driver alone is not enough for Kubernetes CSI snapshots.
   AWS explicitly states that the CSI snapshot controller must be installed before using EBS CSI snapshot functionality.

         volumesnapshots.snapshot.storage.k8s.io
         volumesnapshotcontents.snapshot.storage.k8s.io
         volumesnapshotclasses.snapshot.storage.k8s.io
For EKS, use the supported CSI Snapshot Controller EKS add-on where available rather than manually installing random manifests.

4. Create an EBS VolumeSnapshotClass
    This connects Kubernetes snapshot requests to the EBS CSI driver.

          apiVersion: snapshot.storage.k8s.io/v1
          kind: VolumeSnapshotClass
          metadata:
            name: ebs-csi-snapclass
            labels:
              velero.io/csi-volumesnapshot-class: "true"
          driver: ebs.csi.aws.com
          deletionPolicy: Delete

5. Install Velero with CSI support
   
   Current Velero versions have CSI support integrated; you don't need the old separate velero-plugin-for-csi installation. Velero's documentation says the CSI plugin was merged into Velero starting with release 1.14.


           velero install \
          --provider aws \
          --plugins velero/velero-plugin-for-aws:<version> \
          --bucket my-company-velero-backups \
          --backup-location-config region=us-east-1 \
          --features=EnableCSI
Replace <version> with the Velero AWS plugin version compatible with your Velero version.
