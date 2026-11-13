ETCD BackUp:
=============

* As a k8s admin one of my responsibility is taking a cluster backup.
* we can backup worker loads like pods, deploymets, services, config maps... etc.
* we can also backup the control plane components like etcd, kube-api etc...

1. First way of taking backup:
   ---------------------------
   * we can simpley give a command:

         kubectl get all -A -o yaml > backup.yaml
   * by using the above command we can backup all the resources into a yaml file.
   * This backup.yaml file will only store the configration details.
   * these are just a objects that we are derived from the manifest file.
   * But what about the data in the persistent volumes.
   * This is not a preferd way.

* we manily backup the ectc object in the cluster it will have all the data in the key value pair.

* if you see the controle plane static pods manifests in the /ect/kubernets/manifest folder
* if you describe the ectd manifest file we wll observe few parameters that are passing to etcd while it is creating.

* the main parameter is "*--data-dir=/var/lib/etc*" --where our cluster information will store.

* the another parameter is "*listen-client-urls=the_etcd_liste_urls*"

* few other parameters like:
* "*key_file*"
* "*cert_file*"
* "*ca-cert-file*"
* "*etcd-version*"
* "*mounted-volumes*"

* These volumes mounts we will mount from the host.
* in the host mechine we have two folders one is for certs and another is for data.
* the above two folders we will mount like a volunes in the etcd pod.
* so pod will take the files with in these mounts inside the container.


Now my task is to take the back of the data directry:
----------------------------------------------------

 
Q) How can i do that ?
A) with the help of etcdctl.

* it will helps we us to intract with etcd and perform some admin tasks.

1. we have to install it using apt install etcd-client -y
2. by using etcdctl command we have to pass the env_varible ETCDCTL_API=3

* The major commands in etcdctl is:
* save
* restore
* status
* "*--cacert*"
* "*--cert*"
* "*--endpoints*"
* "*--keys*"

* if you want to set the ETCDCTL_API=3 as env_variable juts pass the export ETCDCTL_API=3 so no need to always give the this env varible in the etcdctl command.

Now we can see the command to take the backup of the etcd:
-----------------------------------------------------------

      ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
        --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
        snapshot save <backup-file-location>



* The below example depicts the usage of the etcdutl tool for verifying a snapshot:

         etcdutl --write-out=table snapshot status snapshot.db

* This should generate an output resembling the example provided below:

        +----------+----------+------------+------------+
        |   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
        +----------+----------+------------+------------+
        | fe01cf57 |       10 |          7 | 2.1 MB     |
        +----------+----------+------------+------------+
        
Note:


      The usage of etcdctl for restoring has been deprecated since etcd v3.5.x and is slated for removal from etcd v3.6. It is recommended to utilize etcdutl instead.



Now we can see the how to restore the backup:
----------------------------------------------


       ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
              --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
              snapshot restore <backup-file-location>/backup_file --data-dir=new_data_path_in_/var/lib/newpath

* onece we run the above commad the backup will be restored in the new path which we have mentioned in the command.
* now we have to update that path in the etcd static pod manifest file.
* we have to modify the volume mounts and volumes.
* so after modified the etcd manifest kubelrt autoimatically restart the etcd pod along with that we have to restart kube-api.
* to restart the kube-api or any other static pods we simply move the manifest files from kubernets/manifest to /temp folder and again move back from /temp to /kubernts
  /manifests folder if do this the static pods will be restarted.
* if you seeing the old config in describe the etcd and we are siing new config in etcd.yaml file then we have to restart kubelet
* command:   "*systemctl restart kubelet*" and "*systemctl daemon-reload*"



 for more refer this: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
