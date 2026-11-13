Custome Resource Defination(CRD'S):
-------------------------------------


Topic:
------

    why we need CRD's in K8S
    What is the use of CRD's

CRD: Custome resource defination

why we need CRD's in K8S
----------------------------

* To extent the k8s api based up on our requirment.
* By applying the CRD k8s will learn about new type of resources and types.
* it will start support CR types in k8s.

Take an example:
-----------------

* if you are deploying an application by writing a deployment.yaml file.
* we will provide all the required configration and we will apply.
* At the time of appling it K8S will validate all the configration with Deployment Resource Defination.
* Simple consider this definations like a templates
* So we cannot add custome filed hear. we can not put image some other palce in yaml file. all these will be mentioned as per the resource defination.


Now we can understand about CRD:
--------------------------------

* if you create any Custome resources k8s will validate it with CRD'S.
* Simple consider this definations like a template.

what this CRD (template) contains ?
-----------------------------------

* It contains Name
* Fields of the custome resources.
* How it shoud be validate
* Data type of those fields
* Once we create the custome resource defination.
* Now we are able to create a Custome resource by using a manifest file/ json files / imparative way
* when we are apply are creating the these CR k8s will validate all the fields in CR with CRD's.

Three Main Objects in when we are creating CR:
-----------------------------------------------

1. CRD
2. CR
3. Custome Controller --> it will manages the lifecycle of Custome resource.

Before know about Custome Controllers:

* first we have to understand what controllers will do in k8s.
* Take deployment controller as an example.
* The Deployment controller will take care of the deployment resource and it lifecycle managment.

* in the same way when we are creating a custome resource we have an option called custome controller.
* we can use that custome resource and ask that controller to manage our custome resource.
* this cutome controller is optional but it is recomended way in prod.
* 


Note: 


  For more deatails refer cks2024-repo---> day -49 folder 



| CRD                                               | CR                               |
| ------------------------------------------------- | -------------------------------- |
| CustomResourceDefinition                          | Custom Resource                  |
| Defines a resource type                           | Actual instance                  |
| Defines the schema                                | Provides values                  |
| Usually created by platform/operator installation | Created by users/apps            |
| `kind: CustomResourceDefinition`                  | `kind: Database`                 |
| Defines `Database`                                | Creates `payment-db`             |
| Defines allowed structure                         | Represents desired configuration |


CRD: 

        apiVersion: apiextensions.k8s.io/v1
        kind: CustomResourceDefinition
        
        metadata:
          name: databases.database.example.com
        
        spec:
          group: database.example.com
        
          names:
            kind: Database
            plural: databases
        
          scope: Namespaced
        
          versions:
            - name: v1
              served: true
              storage: true
        
              schema:
                openAPIV3Schema:
                  type: object
                  properties:
                    spec:
                      type: object
                      properties:
                        engine:
                          type: string
                        version:
                          type: string
                        storage:
                          type: string


Create a CR:


        apiVersion: database.example.com/v1
        kind: Database
        
        metadata:
          name: payment-db
          namespace: production
        
        spec:
          engine: postgres
          version: "16"
          storage: 20Gi

Controller:
-----------

* Controller will watch the desired state what we have mentioned in the CR.yaml file and reconcile it accordingly

* it will identify the diff b/w the actual state and desired state. then it will take action accordingly.

* Desired State: What the user says they want.
* Actual State: What currently exists in the cluster.

* So conceptually: controller will

            Observe
               ↓
            Compare
               ↓
            Act
               ↓
            Observe again

             
* Kubernetes controllers interact with the API Server and receive information about resource changes.

K8S-API Server Responsible for:

        Receiving API requests
        Validation
        Authentication/authorization
        Admission
        Reading/writing cluster state

Controller Responsible for:

        Watching resources
        Understanding desired state
        Comparing with actual state
        Taking corrective action
* Controllers react to both directions of change: at Actual state and desired state.

* Controller Doesn't Just "Create Things"
  
        Create
        Update
        Delete
* Controller Doesn't Directly Change etcd: The controller normally interacts with the Kubernetes API Server.

      Controller → API Server → etcd

* The Word "Reconciliation": 

        Reconciliation means bringing the actual state toward the desired state.

        The controller is not necessarily instantaneous.
        
        There can be a small delay:


* A CRD defines what the custom resource looks like; a CR declares the desired state; a controller gives that resource behavior by continuously reconciling it.
  Controller Does Not Guarantee Instant Correction.

  Custom Controller Internals
  ---------------------------

  * The basic internal flow is:
 
            Kubernetes API Server
                 │
                 │ Watch
                 ▼
              Event
                 │
                 ▼
            Work Queue
                 │
                 ▼
             Reconcile
                 │
                 ▼
        Compare Desired
          vs Actual
                 │
                 ▼
        Take Corrective Action
                 │
                 ▼
        Kubernetes API Server


* There are 4 important pieces:

* Watch: The controller needs to know when something changes.

Conceptually:

        Database Controller
                │
                │ "Tell me when Database objects change"
                ▼
        Kubernetes API Server
        
Instead, it establishes a watch relationship with the Kubernetes API.
So when something happens, the API machinery can notify the controller.


* Event: When a watched resource changes, an event occurs.

For example:

Event 1 — Database created

        payment-db created
               ↓
           CREATE event

Event 2 — Database modified

        storage: 20Gi → 50Gi
               ↓
           UPDATE event

Work Queue: Conceptually, controllers usually place work into a work queue.

The queue might contain:

        payment-db
        orders-db
        inventory-db
The controller worker takes an item from the queue:

        Work Queue
            │
            │ get next item
            ▼
        payment-db
            │
            ▼
        Reconcile(payment-db)

Because many things can happen at the same time so we are using queue.

* Reconcile: The controller takes an item from the queue and calls Reconcile.

For example:

        Queue
          ↓
        payment-db
          ↓
        Reconcile(payment-db)



"What does payment-db want, and what actually exists?"
The API Server itself doesn't exactly "create the event and forward it directly to the controller."

            CR changes
                ↓
          Kubernetes API
                ↓
         Watch notification
                ↓
         Controller receives
             the event
                ↓
           Work Queue
                ↓
            Reconcile
                ↓
       Compare desired vs actual
                ↓
           Take action

For example:

    payment-db
    storage: 20Gi → 50Gi
            ↓
    Controller is notified
            ↓
    Event: payment-db changed
            ↓
    Queue: payment-db
            ↓
    Reconcile(payment-db)
            ↓
    Controller checks current state
            ↓
    PVC needs to be 50Gi
            ↓
    Take corrective action

A change is observed through the watch → the controller receives an event → the resource is placed into the work queue → the controller processes it through reconciliation → it compares desired and actual state → it takes the required action.

* There are three common resource lifecycle events:
  
        | Event    | Meaning                 |
        | -------- | ----------------------- |
        | `CREATE` | A resource was created  |
        | `UPDATE` | A resource was modified |
        | `DELETE` | A resource was deleted  |

* The event triggers reconciliation; the reconciliation logic decides what to do.

Owner References
-----------------

1. First: What problem does OwnerReference solve?

Imagine our Database CR:

    apiVersion: database.example.com/v1
    kind: Database
    metadata:
      name: payment-db
    spec:
      version: "16"
      storage: 20Gi

Our Database Controller creates:

    Database CR
        ↓
    StatefulSet
        ↓
    Pods

Now Kubernetes needs to understand:

Which StatefulSet belongs to which Database?

And:

Which Pods belong to which StatefulSet?

This is where OwnerReference comes in.

What is OwnerReference?

    An OwnerReference is metadata that establishes an ownership relationship between Kubernetes resources.

For example:

    Database CR
         │
         │ owns
         ▼
    StatefulSet

The StatefulSet can contain an OwnerReference pointing to the Database CR.

Conceptually:

    metadata:
      ownerReferences:
        - apiVersion: database.example.com/v1
          kind: Database
          name: payment-db
          uid: <database-uid>

This tells Kubernetes:

    "payment-db is the owner of this StatefulSet."

Parent and Child

It's useful to think of OwnerReferences as:

    Parent
      ↓
    Child

Example:

    Database CR          ← Parent / Owner
         ↓
    StatefulSet           ← Child
         ↓
    Pod                   ← Child

Or with the built-in Deployment example:

    Deployment
        ↓
    ReplicaSet
        ↓
    Pod

The relationships are approximately:

    Deployment
       │
       └── owns ReplicaSet
                 │
                 └── owns Pod

Why can't we just use labels?

Good question.

Labels can tell us:

    labels:
      app: payment

and a selector can find:

    all resources with app=payment

But labels do not establish Kubernetes ownership.

OwnerReference specifically tells Kubernetes:

This object is owned by that object.

        | Labels/Selectors                    | OwnerReference               |
        | ----------------------------------- | ---------------------------- |
        | Helps identify/select resources     | Establishes ownership        |
        | Used by Services, controllers, etc. | Used for ownership/lifecycle |
        | "Which objects match?"              | "Who owns this object?"      |


Garbage Collection

This is one of the biggest reasons OwnerReferences are important.

Suppose:

    Database CR
        ↓ owns
    StatefulSet

Now you delete the Database CR:

    kubectl delete database payment-db

What should happen to the StatefulSet?

    If the StatefulSet is owned by the Database CR, Kubernetes can use garbage collection to clean up dependent resources according to the deletion policy.

Conceptually:

    Delete Database CR
            ↓
    Kubernetes sees OwnerReference
            ↓
    StatefulSet is dependent
            ↓
    Garbage Collection
            ↓
    StatefulSet removed

This prevents orphaned resources.

Without OwnerReference

Imagine:

    Database CR
         ↓
    Controller creates
         ↓
    StatefulSet

But there is no ownership relationship.

Now:

    Delete Database CR
    
    The StatefulSet might remain.

You could end up with:

    Database CR ❌
    StatefulSet  ✅
    Pods         ✅
    PVC          ✅

These are potentially orphaned resources.

That is undesirable.

With OwnerReference

With ownership:

    Database CR
         │
         │ ownerReference
         ▼
    StatefulSet
         │
         ▼
    Pods

When the parent is deleted, Kubernetes can clean up dependents according to garbage-collection behavior.

So:

    Database CR ❌
         ↓
    StatefulSet ❌
         ↓
    Pods ❌

This is one reason OwnerReferences are heavily used by controllers.

Real Deployment Example

You already know:

    Deployment
        ↓
    Deployment Controller
        ↓
    ReplicaSet
        ↓
    ReplicaSet Controller
        ↓
    Pod

OwnerReferences help establish relationships such as:

    Deployment
       │
       │ owns
       ▼
    ReplicaSet
       │
       │ owns
       ▼
    Pod

So Kubernetes knows:

    This ReplicaSet belongs to this Deployment.
    This Pod belongs to this ReplicaSet.

OwnerReference vs Selector

    This is something you should be able to explain in an interview.
    
    Selector
    
    A selector asks:
    
        Which resources match these labels?
    
    Example:
    
        selector:
          matchLabels:
            app: payment
    
    Meaning:
    
        Find resources with:
        app=payment
    
    OwnerReference
    
    OwnerReference asks:
    
        Which resource owns me?
    
    Example:
    
        StatefulSet
        ownerReferences:
          Database/payment-db
    
    Meaning:
    
        StatefulSet belongs to Database/payment-db
    
    Simple memory trick
    
        Selector       → "Who matches me?"
        OwnerReference → "Who owns me?"
    
    OwnerReference contains important information

A typical OwnerReference contains information such as:

    ownerReferences:
      - apiVersion: database.example.com/v1
        kind: Database
        name: payment-db
        uid: 12345678-....
        controller: true
        blockOwnerDeletion: true

uid

    Uniquely identifies that specific object.
    
    This is important because names can potentially be reused.

    Why UID?

    Suppose:
    
    Database payment-db
    UID = ABC
    
    You delete it.
    
    Later you create another:
    
    Database payment-db
    UID = XYZ
    
    Same name, but different object.
    
    OwnerReference uses the UID to identify the exact owner object.
    
    So:
    
    payment-db + UID ABC
    
    is different from:
    
    payment-db + UID XYZ
    
    This prevents ownership ambiguity.



controller: true

    This indicates that this OwnerReference represents the controlling owner of the dependent object.
    
    Example:
    
    Database CR
         ↓
    StatefulSet
    
    The StatefulSet may have:
    
    controller: true
    
    Meaning:
    
    This Database is the controlling owner of this StatefulSet.
    
    A dependent generally has at most one controlling owner.
    
    There can be owner references without being the controlling owner, but we'll keep that distinction simple for now.

Controller + OwnerReference

    Now connect this to everything we've learned.
    
    Suppose:
    
    Database CR
    
    is created.

    The Database Controller:

    Watch Database CR
            ↓
    Event
            ↓
    Queue
            ↓
    Reconcile
            ↓
    Create StatefulSet
            ↓
    Set OwnerReference
    
    So:

    Database CR
         │
         │ ownerReference
         ▼
    StatefulSet
    
    Now the controller has both:
    
    Reconciliation
    "Is the desired state correct?"
    
    and
    
    Ownership
    "Which resources belong to this CR?"

These concepts work together.

blockOwnerDeletion: true

    The dependent indicates that the owner should not be fully deleted until this dependent is handled according to the garbage collector's deletion process.

        OwnerReference
        Who owns me?
        controller: true
        Who is my controlling owner?
        blockOwnerDeletion: true
        Should this dependent participate in blocking
        complete owner deletion during garbage collection?

Garbage Collection & Cascading Deletion


    What is Garbage Collection?
    
    Kubernetes has a mechanism called the Garbage Collector.
    
    Its job includes cleaning up dependent objects when their owners are deleted, based on their ownership relationships and deletion policy.
   
    Cascading Deletion

    When deleting an owner causes its dependent resources to be deleted, we call this cascading deletion.

    Two deletion behaviors you should know

    Kubernetes supports different cascading deletion behaviors.
    
    For now, understand these two:
    
    Background deletion
    
    The owner is deleted first, and Kubernetes garbage collection cleans up dependents afterward.
    
    Conceptually:
    
    Delete Owner
         ↓
    Owner disappears
         ↓
    Garbage Collector
         ↓
    Delete dependents
    Foreground deletion
    
    Kubernetes ensures dependents are deleted before the owner is fully removed.
    
    Conceptually:
    
    Delete Owner
         ↓
    Delete dependents
         ↓
    Dependents gone
         ↓
    Owner fully removed
    
    You don't need to memorize the internal implementation yet.
    
    Just remember:
    
    Background = owner goes away, dependents are cleaned up afterward.
    
    Foreground = dependents are cleaned up before the owner is completely removed.


Day 10 — spec vs status


        apiVersion: database.example.com/v1
        kind: Database
        metadata:
          name: payment-db
        
        spec:
          engine: postgres
          version: "16"
          storage: 20Gi
        
        status:
          phase: Ready
          message: Database is running
        
        
                        Database CR
                           │
                  ┌────────┴────────┐
                  ↓                 ↓
                spec              status
                  │                 │
                  ↓                 ↑
           Desired State      Observed State
                  │                 │
                  └───────┐   ┌─────┘
                          ↓   ↑
                      Controller
                          │
                          ↓
                      Reconcile
                          │
                          ↓
                  Actual Resources
        
        What is spec?
        
        spec describes what the user wants.
        
        For example:
        
        spec:
          engine: postgres
          version: "16"
          storage: 20Gi
        
        The user is saying:
        
        "I want a PostgreSQL 16 database with 20Gi storage."
        
        So:
        
        spec = desired state
        
        Usually, the user or another client changes the spec.
        
        . What is status?
        
        status describes what the controller has observed about the current state.
        
        For example:
        
        status:
          phase: Ready
          message: Database is running
        
        The controller is saying:
        
        "I checked the resources, and the database is currently running."
        
        So:
        
        status = controller-reported observed state
        
        Usually, the controller updates the status.
