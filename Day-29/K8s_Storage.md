 Persistent Volume, Persistent Volume Claim & Storage Class
 ===========================================================

 First we will discuss How volume will work:
 -------------------------------------------

take simple yaml file with empDir{}:
-------------------------------------

apiVersion: v1
kind: Pod
metadata:
  name: redies_pod
spec:
  containers:
    -image: redies
     name: redies_cont
     ports:
       containerPort:
  volumeMounts:
    - name: redies-srorage  ---> storage name
      mountPath: /data/redies ---> path inside a Pod
  volumes:
    -name: redies-srorage
     empDir: {}-----------------------> this a temp storage.if you restart the pod data will gone. it will be persistent lifecycle of the pod not a container.


* Apply the yaml.
* exec into a Pod
* go to /data/redies folder
* create a file with command echo "hello" > file1.txt
* to kill the redies proceess we have to use "ps -aux" command. if it not avalable install with "apt install procps -y"
* by using this ps -aux we will get all the runnig process if you want to kill use command kill -9 process_id.
* if i kill the redies process it will automaticallt come up
* Now you can check the data is available or not in the /data/redies folder.
* we still have our file because hear the container got restarted not a Pod. because hear we mount the volume to Pod not container.
* now if a delete a pod and recreate the pod again then the data will be deleted in the /data/redies.
* because we used empDir hear.



Now we can see how we can over come this issue:
-------------------------------------------------

* we have to make over volume persistent beyond the pod life cycle.

* Firts take an example:

* we have 3 pods
  1. Nginx Pod
  2. redies Pod
  3. node.js pod


* so for all these pods need a Persistent Volume.
* First: for that we will create a persistentVolumeClaim with required memory and access modes --> it just like a request for persistent volume.
  Eg: 10 GI of memory and readwriteAccess
* Second cloud Admin create on Persistent volume with 100GI of memory and with readwriteAccess.
* Third when we are creating the Pod there we have to mentione this persistentvolumeclame and we have to bind with existing persistent volume.
* Then Pod will consume the  required amount if volume from the Persisten Volume based up on the Accessmodes and memory matching.


Now we can see how to create a PV, PVC and Storage Class:
----------------------------------------------------------

* First we have to create PV.
* Next we have to create PVC.
* in the creation of PV we have a policies called Return policies.these ploice tells after deleting the PVS what will happen to PV.
  1. Reatin Police: if the pvc get deleted the pv will remains exit but it will there in release status and no other PVC can request data from it.
  2. Delete: if you delete the PVC the PV will also be deleted.
  3. recycle: if you delete the PVC the PV will be available again for the other PVC to be clamed.
 
Example Yaml file:
-----------------apiVersion: v1
kind: Pod
metadata:
  name: redies_pod
spec:
  containers:
    -image: redies
     name: redies_cont
     ports:
       containerPort:
  volumeMounts:
    - name: redies-srorage  ---> storage name
      mountPath: /data/redies ---> path inside a Pod
  volumes:
    -name: redies-srorage
     persistentVolumeClaim:
       claimName: claim_name


Note: in this section we just use our host mechine as a PV we consueme memory from the host mechineonly because we used single node cluster for it.

* But if you want to do this in Multi Node clusters we need Storage Classes this sc refer the pv in cloud.

* hear we have to typr
  1. static
  2. dynamic volumes
