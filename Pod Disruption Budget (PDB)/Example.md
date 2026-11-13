Current Cluster
==============
You have:
---------
Deployment replicas: 4
Data plane nodes: 2
dpnode-1
---------
pod-1
pod-2

dpnode-2
---------
pod-3
pod-4

PDB:
----
spec:
  minAvailable: 2

This means:
-------------
At least 2 pods must always remain available during a voluntary disruption.

Step 1: Cordon dpnode-1
------------------------
kubectl cordon dpnode-1

Now:

No new pods can be scheduled on dpnode-1.
Existing pods continue running.

Nothing happens to your pods yet.

Step 2: Drain dpnode-1
------------------------
kubectl drain dpnode-1 --ignore-daemonsets

Drain works like this:

For every pod on the node, Kubernetes sends an Eviction API request instead of deleting the pod directly.

Drain
   │
   ▼
Eviction API
   │
   ▼
PDB Check

The PDB controller checks:

"If I evict this pod, will I still satisfy minAvailable?"

First eviction
-----------------
Suppose drain tries to evict pod-1.

Current:

Running Pods = 4
Available Pods = 4

minAvailable = 2

Evicting one pod leaves

Available = 3

Since

3 >= 2

✅ Allowed.

After eviction:
----------------
dpnode-1
---------
pod-2

dpnode-2
---------
pod-3
pod-4

The Deployment immediately notices one replica is missing.

It creates a replacement.

Because dpnode-1 is cordoned,

the new pod can only go to dpnode-2.

dpnode-1
---------
pod-2

dpnode-2
---------
pod-3
pod-4
pod-5

Now again there are 4 running pods.
-----------------------------------
Second eviction

Drain now evicts pod-2.

Before eviction:

Available = 4

After eviction:

Available = 3

Still

3 >= 2

✅ Allowed.

Deployment creates another replacement.

dpnode-2

pod-3
pod-4
pod-5
pod-6

Now dpnode-1 is empty.

Drain completes successfully.

Why doesn't PDB stop this?
----------------------------
Many people think:

"There are only 2 pods on dpnode-1, so PDB should stop drain."

Actually, PDB doesn't care which node the pods are on.

It only checks the total number of available pods.

Initially:

Available = 4
Required = 2

There is room to evict 2 pods.

What if replacement pods take time?

Suppose pod-1 is evicted.

Before pod-5 becomes Ready:

Available = 3

Drain now attempts pod-2.

After eviction:

Available = 2

Still satisfies

minAvailable = 2

So Kubernetes allows it.

When would drain pause?
--------------------------
Imagine instead your PDB was:

minAvailable: 3

Initially:

Available = 4

Drain evicts pod-1.

Available = 3

Allowed.

Now it immediately tries pod-2.

If pod-5 is not Ready yet, then:

Available = 2

But PDB requires:

minAvailable = 3

❌ Eviction is denied.

Drain waits until the replacement pod becomes Ready:

pod-5 Ready

Available = 4

Now it retries eviction of pod-2, which is allowed.

Timeline for minAvailable: 2
--------------------------------
Initial

dpnode-1        dpnode-2
---------       ---------
pod1            pod3
pod2            pod4

Available = 4

        │
        ▼
Evict pod1

Available = 3
PDB OK

        │
        ▼
Deployment creates pod5

dpnode-2
---------
pod3
pod4
pod5

Available = 4

        │
        ▼
Evict pod2

Available = 3
PDB OK

        │
        ▼
Deployment creates pod6

dpnode-2
---------
pod3
pod4
pod5
pod6

Upgrade node
Key takeaway

The PDB is evaluated for each eviction request, not for the entire node drain at once.
With replicas: 4 and minAvailable: 2, Kubernetes can safely evict both pods from dpnode-1 because evicting each pod never reduces the number of available pods below 2. 
If replacement pods are slow to become Ready and an eviction would violate the PDB, the drain command simply waits and retries once enough pods are available again.

This is one of the reasons Kubernetes upgrades are performed one node at a time—it allows workloads to move to the remaining nodes while the PDB ensures your application's availability isn't reduced below the configured threshold.
