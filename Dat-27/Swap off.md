1. First: What is swap?
   --------------------
Linux has two types of memory storage:

RAM
 ↓
Fast memory


Disk
 ↓
Slow storage

When RAM becomes full, Linux can move some memory pages from RAM to disk. This disk area is called swap.

For example:

RAM = 4 GB
Swap = 2 GB


RAM gets full
     ↓
Linux moves some memory to swap
     ↓
Disk is used as extra memory

This is useful for normal Linux systems because it prevents applications from immediately running out of memory.


2. Why does Kubernetes care about swap?
   -----------------------------------
Kubernetes needs to make decisions about how much memory is available to containers and pods.

Suppose you have:

Node RAM = 4 GB


Pod A → 1 GB
Pod B → 1 GB
Pod C → 1 GB

Kubernetes uses cgroups to control and account for container resources.

3. Now your first command
   -----------------------
swapoff -a

This means:

Turn off all currently active swap.

-a means all swap areas.

You can check before/after with:

swapon --show

or:

free -h

Before:

               total   used   free
RAM            4Gi     2Gi    2Gi
Swap           2Gi     0Gi    2Gi

After:

               total   used   free
RAM            4Gi     2Gi    2Gi
Swap           0B      0B     0B

So:

swapoff -a
     ↓
Currently active swap is disabled

For example:

Pod A
  memory limit = 1 GB


Pod B
  memory limit = 1 GB

Kubernetes expects memory management to be predictable.

But if Linux starts moving container memory to swap:

RAM
 |
 +---- Pod A
 |
 +---- Pod B
 |
 ↓
Swap
(Disk)

memory behavior becomes less predictable.

That's why traditional kubeadm setups disable swap.

4. But there is a problem
   ----------------------
Suppose you run:

swapoff -a

The swap is disabled right now.

But after reboot, Linux may read /etc/fstab and enable swap again.

For example:

/etc/fstab


/dev/mapper/swap   none   swap   sw   0 0

During boot:

Server reboot
    ↓
Linux reads /etc/fstab
    ↓
Finds swap configuration
    ↓
Swap gets enabled again

Then your Kubernetes node has swap enabled again.

5. That's why we use the second command
   ------------------------------
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

This modifies /etc/fstab.

Its purpose is:

Comment out the swap entry so that swap doesn't automatically come back after reboot.

For example, before:

/dev/mapper/swap none swap sw 0 0

After:

#/dev/mapper/swap none swap sw 0 0

The # means the line is commented out.

So Linux won't use that line during boot.

6. Why do we need BOTH commands?
   ------------------------------
This is the important part.

Command 1
swapoff -a

Disables swap right now.

Command 2
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

Prevents swap from being automatically enabled after reboot.

So:

             Kubernetes Node
                    |
          ---------------------
          |                   |
      swapoff -a          edit /etc/fstab
          |                   |
      disable now       disable permanently
          |                   |
          └────────┬──────────┘
                   ↓
             Swap remains OFF
7. Why Kubernetes traditionally required this
   -------------------------------
Kubernetes uses:

kubelet
   ↓
cgroups
   ↓
container resources

For example, you might configure:

resources:
  requests:
    memory: "512Mi"
  limits:
    memory: "1Gi"

Kubernetes expects the node's memory accounting and eviction behavior to be predictable.

With traditional swap disabled:

Node
 |
 +--- RAM
 |
 +--- cgroups
       |
       +--- Pod A
       +--- Pod B
       +--- Pod C

Memory pressure can be handled using Kubernetes' mechanisms, including pod eviction and OOM behavior, rather than silently paging workloads to disk.

8. One important modern Kubernetes detail
   -----------------------------------------
You may see newer Kubernetes documentation discussing swap support.

That's because newer Kubernetes versions can be configured to use swap under certain conditions.

So don't memorize:

"Kubernetes can NEVER use swap."

The more accurate interview answer is:

Traditionally, kubeadm installations disable swap because kubelet expects predictable memory management and historically required swap to be disabled.
Modern Kubernetes versions have controlled swap support, but unless you intentionally configure and validate swap behavior, disabling swap remains the common kubeadm setup.

9. Easy way to remember
   ---------------------
Think of it like this: 

RAM = Kubernetes' controlled workspace

Swap = Linux's emergency overflow area

Kubernetes wants predictable memory management, so traditional kubeadm setup says:

swapoff -a

"Turn swap off now."

And:

sed ... /etc/fstab

"Don't turn it back on after reboot."

Final flow
              kubeadm
                 |
                 ↓
              kubelet
                 |
                 ↓
          Memory management
                 |
        -------------------
        |                 |
       RAM              Swap
        |                 |
     Controlled       Disabled
     by cgroups

So the real reason is not that swap is "bad"; it's that Kubernetes wants predictable and controllable memory management, and the traditional kubeadm configuration achieves that by disabling swap.
