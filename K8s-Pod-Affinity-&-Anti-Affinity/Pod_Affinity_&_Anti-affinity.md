Pod Affinity and Pod Anti-Affinity in Kubernetes
=================================================

First, understand the concept:

- Pod Affinity: Schedule a Pod on a node where a matching Pod already exists, or where matching Pods exist within a specified topology.
- Pod Anti-Affinity: Prefer or require scheduling a Pod away from nodes or topology domains containing matching Pods.

Pod Affinity example:
-----------------------

Imagine you have three worker nodes:

    Worker-1: backend Pod
    Worker-2: empty
    Worker-3: empty


* You want the frontend Pod to be scheduled on a node where the backend Pod is already running.

You can use Pod Affinity:

    apiVersion: v1
    kind: Pod
    metadata:
      name: frontend
      labels:
        app: frontend
    spec:
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  app: backend
              topologyKey: kubernetes.io/hostname
      containers:
        - name: frontend
          image: nginx


How it works:

    - podAffinity enables Pod Affinity.
    - labelSelector looks for Pods labelled app: backend.
    - topologyKey: kubernetes.io/hostname means the matching backend Pod must be on the same node as the frontend.
    - requiredDuringSchedulingIgnoredDuringExecution makes this a hard scheduling requirement.

Result:

    Worker-1: backend + frontend  ✅
    Worker-2: empty
    Worker-3: empty

Pod Anti-Affinity example:
----------------------------

* Now imagine you run three replicas of a payment application.

* You don't want matching payment Pods on the same worker node because a node failure could affect multiple replicas.

      apiVersion: v1
      kind: Pod
      metadata:
        name: payment
        labels:
          app: payment
      spec:
        affinity:
          podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              - labelSelector:
                  matchLabels:
                    app: payment
                topologyKey: kubernetes.io/hostname
        containers:
          - name: payment
            image: nginx


How it works:

    - podAntiAffinity keeps matching Pods apart.
    - labelSelector finds other Pods labelled app: payment.
    - topologyKey: kubernetes.io/hostname applies the separation rule per node.
    - requiredDuringSchedulingIgnoredDuringExecution means Kubernetes must not place matching Pods on the same node.

Result with three worker nodes:
  
    Worker-1: payment-1  ✅
    Worker-2: payment-2  ✅
    Worker-3: payment-3  ✅


If there are only two suitable nodes for three replicas, one Pod may remain Pending.

Understand the two scheduling options:
--------------------------------------

<img width="1357" height="392" alt="image" src="https://github.com/user-attachments/assets/bbe9959b-81b2-49c0-8f39-b27f2cc437b6" />



If no node satisfies the rule, the frontend Pod stays Pending.
