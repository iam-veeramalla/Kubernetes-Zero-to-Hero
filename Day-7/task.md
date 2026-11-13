this exercise, you will create a Deployment and expose a container port for its Pods. You will demonstrate the differences between the service types ClusterIP and NodePort.

Note

      If you do not already have a Kubernetes cluster, you can create a local Kubernetes cluster by following Day06 Video Also, do the node binding at the cluster level if you are using KIND. The Day9 video has the details on how to do that.

Task details
-------------
Create a Service named myapp of type ClusterIP that exposes port 80 and maps to the target port 80.

Create a Deployment named myapp that creates 1 replica running the image nginx:1.23.4-alpine. Expose the container port 80.

Scale the Deployment to 2 replicas.

Create a temporary Pod using the image busybox and run a wget command against the IP of the service.

     apiVersion: v1
     kind: Pod
     metadata:
       name: busy-box-pod
       labels:
         app: busy-box
     spec:
       containers:
         - name: busy-box-cont
           image: busybox
           command: ["sleep"]
           args: ["900"]
           ports:
             - containerPort: 80

Enter in to the container:

      kubectl exec -it pod_name -- /bin/sh

Ping the ClusterIp service inside the Pod:

     wget -o- http://cluster_ip:service_port

you we will get the responce.


Run a wget command against the service outside the cluster.


            wget -o- http://cluster_ip:service_port

you will not get the responce.

Change the service type so the Pods can be reached outside the cluster.


    ClusterIP ---> NodePort

Run a wget command against the service outside the cluster.

Firts:
get the kind node ip where our pods are running by using:


     kubectl get nodes -o wide

pick the node Ip and run the cmd out side the cluster:

    wget -o- http://node_ip:node_port


Discuss: Can you expose the Pods as a service without a deployment?

Discuss: Under what condition would you use the service types LoadBalancer, node port, clusterIP, and external?

Share your learnings: Document your key takeaways and insights in a blog post and social media update

Make it public: Share what you learn publicly on LinkedIn or Twitter.

Tag us and use the hashtag: Include the following in your post:
Tag @PiyushSachdeva and @CloudOps Community (on both platforms)
Use the hashtag #40daysofkubernetes
Embed the video: Enhance your blog post by embedding the video lesson from the Kubernetes series. This will give you visual context and reinforce your written explanations.
Blog Post Focus 📝
Clarity is essential: Write your blog post clearly and concisely, making it easy for anyone to grasp the concepts, regardless of their prior Kubernetes experience.
