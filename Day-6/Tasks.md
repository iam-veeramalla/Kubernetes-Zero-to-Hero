In this exercise, you will create a Deployment with multiple replicas. After inspecting the Deployment, you will update its Pod template. You will also be able to use the rollout history to roll back to a previous revision.
==
   Note:

   If you do not already have a Kubernetes cluster, you can create a local Kubernetes cluster by following Day06 Video

Replicaset
----------
Create a new Replicaset based on the nginx image with 3 replicas
Update the replicas to 4 from the YAML
Update the replicas to 6 from the command line

Deployment
----
1. Create a Deployment named nginx with 3 replicas. The Pods should use the nginx:1.23.0 image and the name nginx. The Deployment uses the label tier=backend. The Pod template should use the label app=v1.
2. List the Deployment and ensure the correct number of replicas is running.
3. Update the image to nginx:1.23.4.
4. Verify that the change has been rolled out to all replicas.
5. Assign the change cause "Pick up patch version" to the revision.
6. Scale the Deployment to 5 replicas.
7. Have a look at the Deployment rollout history.
8. Revert the Deployment to revision 1.
9. Ensure that the Pods use the image nginx:1.23.0.
    
Troubleshooting the issue
-----------------------
1. Apply the below YAML and fix the issue with it

apiVersion: v1
kind:  Deployment
metadata:
  name: nginx-deploy
  labels:
    env: demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
  replicas: 3
  selector:
    matchLabels:
      env: demo
      
2. Apply the below YAML and fix the issue with it
apiVersion: v1
kind:  Deployment
metadata:
  name: nginx-deploy
  labels:
    env: demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
  replicas: 3
  selector:
    matchLabels:
      env: dev
3. Share your learnings: Document your key takeaways and insights in a blog post and social media update
4. Make it public: Share what you learn publicly on LinkedIn or Twitter.
   * Tag us and use the hashtag: Include the following in your post:
     Tag @PiyushSachdeva and @CloudOps Community (on both platforms)
      Use the hashtag #40daysofkubernetes
      Embed the video: Enhance your blog post by embedding the video lesson from the Kubernetes series. This will provide visual context and reinforce your written explanations.
     
Blog Post Focus 📝
-------------
Clarity is essential: Write your blog post clearly and concisely, making it easy for anyone to grasp the concepts, regardless of their prior Kubernetes experience.
