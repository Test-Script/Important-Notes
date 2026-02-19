==========================================================================================================
                                        Kubernetes Architecture
==========================================================================================================

Understanding Of Kubernetes In Depth

1. Control Plane (Management Plane)
2. Data Plane (Worker Nodes)

==========================================================================================================
                                            Management Plane
==========================================================================================================

1. kube-apiserver

    - Entry point for all REST operations.
    - Exposes Kubernetes API (CRUD on objects).
    - AuthN, AuthZ, Admission Control.
    - Stateless component (can scale horizontally).
    - All components communicate only via API server.

    Every write operation:

    Immediately persisted to etcd.

    Every read:

    Served from cache (watch cache) when possible.

    etcd is not hit for every single read.

2. etcd

    - Distributed key-value store.
    - Stores cluster state (pods, deployments, secrets, configmaps).
    - Strong consistency (Raft consensus).
    - Must be backed up regularly (critical component).

    In-Depthe Understading of Kubelet,

    1. How data flows inside Kubernetes

        - etcd = the “source of truth” : 
            - etcd stores all cluster state data — nodes, pods, deployments, secrets, configmaps, RBAC roles, everything.
            - Any change in the cluster (like creating a Pod, updating a Node status, deleting a Service) is recorded in etcd.
        - API Server = the central gateway : 
            - All other components — kubelet, scheduler, controller manager, kubectl, etc. — never talk to each other directly.
            - They all communicate only via the API Server.
            - The API Server reads/writes data to etcd and provides that data to the components that need it.
        - Flow example: 
            - User (or controller) sends a request → kubectl apply -f pod.yaml.
            - API Server validates & stores the Pod object → in etcd.
            - Scheduler watches the API Server → sees an unscheduled Pod → chooses a Node.
            - Scheduler updates the Pod object (with nodeName) → API Server → etcd.
            - Kubelet watches the API Server → sees a Pod assigned to its node → starts the containers.
            - Kubelet reports Pod status back → API Server → etcd.

        Every operation in Kubernetes goes through the API Server and gets persisted in etcd.
        Other components just watch the API Server for changes instead of querying etcd directly.

3. kube-scheduler

    - Watches for unscheduled Pods.
    - Selects optimal node based on:
        - Resource requests/limits
        - Taints & tolerations
        - Node affinity/anti-affinity
        - Topology spread constraints

    Frequency of Schedular to report to API Server:

    kube-scheduler → API Server

    - Scheduler does NOT poll periodically.

        It uses:

            ✔ Informers
            ✔ Watch API
            ✔ Event-driven queue

        Flow:

        - Scheduler establishes a long-lived WATCH connection to API Server.

        API Server pushes updates when:

        - New Pod created

        - Node updated

        - Pod deleted

        - Taints changed

        Scheduler immediately reacts.

        So frequency = real-time event driven.

4. kube-controller-manager

    - Runs multiple controllers:
        - Deployment controller
        - ReplicaSet controller
        - Node controller
        - Job controller
        - Endpoint controller

        These controllers implement the reconciliation loop: Desired State ≠ Current State → Take corrective action.
    
    kube-controller-manager → API Server

    Same model as scheduler.

    - Uses shared informers

    - Maintains watch streams

    - Reacts to events

    Example:

    Deployment created → ReplicaSet controller reacts immediately

    Node NotReady → Node controller reacts

==========================================================================================================
                                        Worker Node (Data Plane)
==========================================================================================================

Worker nodes run actual workloads.

1. kubelet

    - Node agent.
    - Communicates with API server.
    - Ensures containers defined in PodSpec are running.
    - Performs:
        - Health checks
        - Volume mounts
        - Secret injection

    In-Depthe Understading of Kubelet,

    1. Kubelet reports node status to the API Server.

        - Every node runs a kubelet agent.
        - The kubelet periodically collects resource usage and capacity information from the node (via  cAdvisor and the OS).
        - It sends this data (CPU, memory capacity, allocatable, and current usage) to the Kubernetes API Server as part of the Node object status and Pod status.

    2. API Server stores that data in etcd

        - The API Server updates the cluster state in etcd, which is the single source of truth.

    3. Scheduler reads node resource info from the API Server

        - The Kube-scheduler does not communicate directly with the kubelet.
        - It queries the API Server to get the latest node resource data (which originally came from kubelet).
        - Then it decides on which node a new Pod should be scheduled, based on resource availability, taints/tolerations, affinity, etc.

    Frequency Of Node Status Update, and Heatbeat Signal.

    kubelet → API Server
    
    1. Node Status Updates

        - Default: every 10 seconds

        Controlled by: --node-status-update-frequency (default 10s)

        - This updates:

            - Capacity

            - Allocatable

            - Conditions (Ready, MemoryPressure, DiskPressure, etc.)
        
    2. Node Lease (HeartBeat)

        - Every 10 Seconds
        - Uses a lightweight Lease object in kube-node-lease namespace
        - Faster and cheaper than full Node object updates

    If kubelet stops updating:

        After 40 seconds (default) node becomes NotReady
        (--node-monitor-grace-period in controller-manager)

2. kube-proxy

    - Implements Service networking.
    - Manages iptables/ipvs rules.
    - Enables:
        - ClusterIP
        - NodePort
        - LoadBalancer
    - Provides internal load balancing.

3. Container Runtime

    - containerd
    - CRI-O
    - Responsible for:
        - Pulling images
        - Creating containers
        - Managing container lifecycle

==========================================================================================================
                                            Kubelet
==========================================================================================================

    On-demand only.

    When you run:
    - kubectl get pods

    It makes a request once.
    Unless you use:
    kubectl get pods -w
    That establishes a watch stream.


===========================================================================================================
                                Component | Communication Pattern | Frequency
===========================================================================================================

1. kubelet (node status) | Update | ~10s
2. kubelet (lease heartbeat)| Update | ~10s
3. scheduler | Watch stream | Real-time
4. controller-manager | Watch stream | Real-time
5. kubectl | On demand | User-driven

==========================================================================================================
                                Core Kubernetes control-plane engineering.
==========================================================================================================

“Kubernetes components use watch-based, event-driven reconciliation loops against the API Server rather than periodic polling.”

Three precise sections:

    1. How Informers work internally

    2. What happens when a watch connection drops

    3. How API Server scales with thousands of watches

Excellent — this is **core Kubernetes control-plane engineering**.
I’ll explain this in three precise sections:

1. How Informers work internally
2. What happens when a watch connection drops
3. How API Server scales with thousands of watches

---

# How Informers Work Internally

Informers are part of **client-go** and are used by:

    * kube-scheduler
    * kube-controller-manager
    * Custom controllers
    * Operators

They are built on top of:

    * List API
    * Watch API
    * Local cache
    * Workqueue

## Internal Flow of an Informer

Step 1 — Initial LIST
The informer calls the API Server:

```
GET /api/v1/pods
```

It retrieves all existing objects and stores them in a local cache (Store / Indexer).

Step 2 — Establish WATCH
Then it calls:

```
GET /api/v1/pods?watch=true&resourceVersion=X
```

Now it opens a **long-lived HTTP connection**.

Step 3 — API Server streams events

Events streamed are:

    * ADDED
    * MODIFIED
    * DELETED
    * BOOKMARK (progress notification)

Step 4 — Delta FIFO Queue

Events are pushed into a **DeltaFIFO queue**:

    * Deduplicates events
    * Maintains order

Step 5 — Local Cache Update

The informer updates its local in-memory cache.

Step 6 — Event Handlers Triggered

Registered handlers run:

    ```
    OnAdd(obj)
    OnUpdate(oldObj, newObj)
    OnDelete(obj)
    ```

Step 7 — Object added to Workqueue

Controller processes object from queue and reconciles desired vs actual state.

## Why Informers Are Efficient

Without informers:

    * Every controller would repeatedly query API Server.

With informers:

    * One LIST
    * One WATCH
    * Everything local afterward
    * Event-driven updates

This drastically reduces API load.

# What Happens When Watch Connection Drops?

Watch connections can drop due to:

    * API Server restart
    * Network glitch
    * Load balancer timeout
    * etcd compaction
    * Idle timeout

## Case 1: Normal Drop

client-go automatically:

    1. Detects connection closed
    2. Re-establishes LIST + WATCH
    3. Continues from last known resourceVersion

No data loss.

---

## Case 2: "ResourceVersion Too Old" Error

etcd compacts old revisions.

If the informer tries to resume from an expired version:

API Server returns:

```
410 Gone
resourceVersion too old
```

Then informer does:

1. Full LIST again
2. Gets fresh resourceVersion
3. Starts new WATCH

This ensures consistency.

### Important Concept: ResourceVersion

Every Kubernetes object has:

```
metadata.resourceVersion
```

It represents the etcd revision.

Watches always resume from a specific resourceVersion.

That’s how Kubernetes guarantees no missed events.

## How API Server Scales with Thousands of Watches

Large clusters can have:

    * 5,000 nodes
    * 200,000 pods
    * Thousands of controllers
    * Many kubectl users
    * Operators

That means thousands of open watch streams.

## How does API Server handle this?

Mechanism 1: Watch Cache

The API Server does NOT hit etcd for every watch.

Instead:

    * It maintains an in-memory watch cache per resource type.
    * New watch clients attach to the cache.
    * Events are served from memory.

This drastically reduces etcd load.

Mechanism 2: Shared Informers

Inside components like controller-manager:

Multiple controllers share the same informer.

Example:

    * ReplicaSet controller
    * Deployment controller

They don’t create separate watches for Pods.
They share one.

This reduces watch count.

Mechanism 3: API Priority & Fairness (APF)

API Server uses:

    * Request queues
    * Flow control
    * Priority levels

To prevent overload.

High priority:

    * System controllers
    * kubelet

Lower priority:

    * kubectl users

This protects control plane stability.

Mechanism 4: Horizontal Scaling of API Server

In production clusters:

    * Multiple API Server replicas run
    * Behind a load balancer
    * Stateless design

Watches are distributed across instances.

Mechanism 5: etcd Compaction & Efficient Storage

    * etcd periodically compacts old revisions.
    * Prevents unbounded growth.
    * Improves watch performance.


## "Kubernetes uses a LIST-WATCH mechanism implemented via client-go informers. Informers maintain a local cache synchronized through long-lived watch streams. If a watch drops, it resumes using resourceVersion. The API Server scales using watch caches, shared informers, API priority and fairness, and horizontal replication."

If you want, next we can go even deeper into:

* DeltaFIFO internals
* Workqueue rate limiting
* Scheduler framework internals
* How kubelet watches PodSpec changes
* etcd revision model in detail

You are now in advanced Kubernetes control-plane territory.


==========================================================================================================
            Informers using a real production-style scenario so the mechanism becomes concrete.
==========================================================================================================

Example : A Deployment creates Pods, and the ReplicaSet controller must maintain desired replicas.

This is exactly how Kubernetes controllers work internally.

Real-Time Scenario:

kubelet Client : kubectl apply -f deployment.yaml

    Deployment spec:

        replicas: 3
        image: nginx

    Now let’s see how informers make everything event-driven.

Step 1️⃣ — Object Created in API Server

    API Server validates Deployment.

    Stores it in etcd.

    Deployment controller detects it.

But how does it detect it?

👉 Using an Informer.

🔎 What the ReplicaSet Controller Actually Does

Inside kube-controller-manager:

There is a shared informer for:

    Deployments

    ReplicaSets

    Pods

Let’s focus on the Pod informer.

📦 How the Pod Informer Works (Internally)

    Phase 1 — Initial LIST

    When controller starts:

    It does:

    GET /api/v1/pods


    API Server returns:

    All existing Pods

    With a resourceVersion (say 10500)

    The informer:

    Stores them in an in-memory cache (Indexer)

    Now it has a local snapshot.

    Phase 2 — Start WATCH

    Informer then does:

    GET /api/v1/pods?watch=true&resourceVersion=10500


    This opens a long-lived HTTP connection.

    Now the API Server streams events:

        ADDED

        MODIFIED

        DELETED

    No polling.

    Pure push model.

🎯 Real-Time Event Flow Example

Now Deployment creates 3 Pods.

Pod1 created → API Server stores it → resourceVersion 10501

API Server pushes event to all watchers:

#    Event: ADDED
#   Object: Pod1
#    resourceVersion: 10501

Informer receives it immediately.

🔁 What Happens Inside the Informer

    This is the internal pipeline:

    Event received

    Put into DeltaFIFO queue

    Update local cache

    Trigger event handler

    Add object key to workqueue

    Let’s visualize:

        API Server
            ↓
        Watch Stream
            ↓
        Reflector
            ↓
        DeltaFIFO
            ↓
        Indexer (local cache)
            ↓
        Event Handler
            ↓
        WorkQueue
            ↓
        Controller Reconcile()

🧠 What is Reflector?

    Reflector is the component that:

    Does LIST

    Starts WATCH

    Keeps them in sync

    It ensures cache is always up to date.

🧠 What is DeltaFIFO?

    DeltaFIFO:

    Ensures ordering

    Deduplicates events

    Tracks object changes

    Example:

    If Pod updated 5 times quickly:

    It merges them.

    Controller processes latest state.

    Prevents unnecessary reconciliation.

🔄 Controller Reconciliation

    Now controller’s worker picks item from WorkQueue.

    It compares:

    Desired State:

    replicas = 3

    Actual State:

    count of Pods in cache

    If less than 3:
    → create more Pods

    If more than 3:
    → delete extra Pods

    This loop continues forever.

    This is called:

    Reconciliation loop

⚡ Why Informers Are Powerful

    Without informers:

    Controller would do:

        while true:
        GET /pods
        sleep(5)


    This would destroy API Server under load.

With informers:

    One LIST

    One WATCH

    Everything local in memory

    Near real-time reaction

🔬 What Happens If Watch Drops?

    Example:

    Network glitch.

    Watch connection closed.

    Reflector:

    Detects error

    Re-LIST from last resourceVersion

    Starts new WATCH

    If resourceVersion expired:

    Gets 410 Gone

    Does full LIST

    Starts fresh WATCH

    No events lost.

🏗 Scaling Example (Large Cluster)

    Imagine:

        10,000 Pods

        50 controllers

    Without shared informer:

        50 LIST

        50 WATCH connections

    With shared informer:

        1 LIST

        1 WATCH

        Shared cache

        Huge performance gain.

🎯 Real-World Analogy

    Think of Informer like:

📡 A live stock market ticker

Instead of:

Refreshing the website every second

You:

Open one live streaming connection

Server pushes updates instantly

Informer = live subscription model

🧩 Technical Aspect whre you can understand it very well

Informers use a LIST-WATCH mechanism implemented via a Reflector. The Reflector populates a local cache and maintains synchronization using watch streams. Events are stored in a DeltaFIFO queue and processed via event handlers and a rate-limited workqueue, enabling efficient event-driven reconciliation.

🔎 In One Sentence

Informer =

LIST once → WATCH continuously → update local cache → trigger reconciliation.

==========================================================================================================
                                End to End Workflow (Pod Creation)
==========================================================================================================

1. User applies YAML → kubectl apply
2. Request goes to kube-apiserver
3. Object stored in etcd
4. Scheduler assigns node
5. kubelet on selected node:
    - Pulls image
    - Creates Pod via container runtime
6. kube-proxy configures networking
7. Pod becomes Running

==========================================================================================================
                                        Kubernetes Networking Model
==========================================================================================================

- Every Pod gets a unique IP.

- Pod-to-Pod communication is flat (no NAT).

- Service provides stable virtual IP.

- DNS-based service discovery via CoreDNS.

==========================================================================================================
                                        High Availability Architecture
==========================================================================================================

- Production clusters typically have:
    - 3+ control plane nodes
    - etcd cluster (3 or 5 nodes)
    - Multiple worker nodes
    - External Load Balancer in front of API server

==========================================================================================================
                                        Key Architectural Principles
==========================================================================================================

- Declarative configuration

- Self-healing

- Horizontal scalability

- Loosely coupled components

- API-driven architecture

- Controller-based reconciliation model

Summary Table
Component | Communication Pattern | Frequency

kubelet (node status) | Update | ~10s
kubelet (lease heartbeat)| Update | ~10s
scheduler | Watch stream | Real-time
controller-manager | Watch stream | Real-time
kubectl | On demand | User-driven

==========================================================================================================