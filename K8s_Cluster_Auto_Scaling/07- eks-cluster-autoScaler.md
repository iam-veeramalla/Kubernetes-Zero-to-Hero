
---
# **EKS Cluster Autoscaler – Overview and Implementation Guide**
---

## **What is EKS Cluster Autoscaler?**  
EKS Cluster Autoscaler is a Kubernetes component that automatically adjusts the number of worker nodes in an **Amazon EKS** cluster based on workload demands. It ensures that sufficient compute resources are available while also optimizing costs by removing unused nodes when they are no longer needed.

### **Why Use EKS Cluster Autoscaler?**
- **Scales Up Nodes**: Adds new worker nodes when the cluster runs out of resources.
- **Scales Down Nodes**: Removes underutilized nodes to save costs.
- **Improves Performance**: Ensures applications have enough compute capacity.
- **Optimizes Costs**: Removes idle resources automatically.

---

## **Step-by-Step Implementation of EKS Cluster Autoscaler**

### **Step 1: Describe the Node Group in EKS**
To check the existing node group details, run:
```sh
aws eks describe-nodegroup --cluster-name <cluster-name> --nodegroup-name <node-group-name>
```
This command provides information about the current node group, including its scaling configuration, IAM roles, and status.

---

### **Step 2: Attach IAM Policy for Cluster Autoscaler**  
EKS worker nodes need proper IAM permissions to manage Auto Scaling Groups (ASG). Attach the **AmazonEKSClusterAutoscalerPolicy** to the IAM role of your worker nodes.

---

### **Step 3: Install Cluster Autoscaler using Helm**  
Deploy Cluster Autoscaler using Helm, which simplifies the installation process.

#### **Run the following commands:**
```sh
helm repo add autoscaler https://kubernetes.github.io/autoscaler
helm repo update
helm install cluster-autoscaler autoscaler/cluster-autoscaler \
  --namespace kube-system \
  --set autoDiscovery.clusterName=<cluster-name> \
  --set awsRegion=<region-code> \
  --set extraArgs.balance-similar-node-groups=true \
  --set extraArgs.skip-nodes-with-system-pods=false
```

**Explanation:**
- **`autoDiscovery.clusterName=<cluster-name>`**: Enables automatic node discovery for the specified EKS cluster.
- **`awsRegion=<region-code>`**: Set your AWS region, e.g., `ap-south-1` (Mumbai).
- **`balance-similar-node-groups=true`**: Distributes workloads evenly across node groups.
- **`skip-nodes-with-system-pods=false`**: Ensures system pods do not block node termination.


#### **Verify Cluster Autoscaler Deployment**
Check if the Cluster Autoscaler pod is running:
```sh
kubectl get pods -n kube-system | grep cluster-autoscaler
```

---

### **Step 4: Deploy `php-apache` Application and Increase Replicas**
Now, deploy a sample application (`php-apache`) and increase the replica count to test autoscaling.

#### **Scale Up the Application**
```sh
kubectl scale deployment php-apache --replicas=50
```
- This command increases the number of running pods from the default count to **50**.
- If the current worker nodes **do not have enough resources**, Cluster Autoscaler will scale up and add more nodes.

---

### **Step 5: Verify Logs and Autoscaler Behavior**
Monitor the logs of the deployment and ensure that autoscaling is happening.

#### **Check Application Logs**
```sh
kubectl logs -f deployment/php-apache
```

#### **Monitor Cluster Autoscaler Logs**
```sh
kubectl logs -f -n kube-system deployment/cluster-autoscaler
```
- If scaling is working correctly, you will see messages like **"Expanding node group"** in the logs.
- If it does not scale up, check for errors in the **Cluster Autoscaler** logs.

---
