When I started learning Kubernetes, everyone kept saying "etcd is the brain of the cluster." 🤔
But I had one question...
How does etcd make sure every control plane has the same data?
After spending some time understanding the Raft Consensus Algorithm, it finally made sense.

Here's a simple explanation:
Imagine a Kubernetes cluster with 3 control plane nodes and 20 worker nodes
There are 3 etcd members:
 👑 1 Leader
 👥 2 Followers
Here's what happens:
👉 Kubernetes sends every write request (kubectl apply -f pod.yaml) to the etcd Leader.
👉 The Leader then replicates that data to all the Follower nodes.
👉 Once the majority of followers acknowledge the change, the data is committed.
That's it! The write is now considered successful. ✅

The interesting part is that followers never accept write requests directly. They simply replicate the Leader's data.

❌ And if the Leader goes down?
 No manual intervention is needed.
🗳️ Raft Consensus automatically elects a new 👑 Leader, allowing Kubernetes to continue working with minimal disruption.

If your cluster has:
• 3 etcd nodes → Minimum 2 must be healthy.
 • 5 etcd nodes → Minimum 3 must be healthy.
 • 7 etcd nodes → Minimum 4 must be healthy.

That's why production Kubernetes clusters usually use an odd number of etcd members (3, 5, or 7 multi cluster).

In self-managed Kubernetes, we can see and manage the etcd cluster ourselves., But in Amazon EKS, things are different.
AWS manages the entire control plane for us, including:
 ✅ Kubernetes API Server
 ✅ Scheduler
 ✅ Controller Manager
 ✅ etcd cluster
So, we don't SSH into etcd or take manual backups like we would in a kubeadm cluster.
✅ EKS Control plane distributed across at least three Availability Zones (for regional clusters)

Sometimes, the most complex Kubernetes concepts become easy once you understand the flow instead of just memorizing definitions.
I created the below visual because it helped me understand how etcd works internally. Hopefully, it helps someone else too. 😊
