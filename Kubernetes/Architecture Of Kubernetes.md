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

4. kube-controller-manager

    - Runs multiple controllers:
        - Deployment controller
        - ReplicaSet controller
        - Node controller
        - Job controller
        - Endpoint controller

        These controllers implement the reconciliation loop: Desired State ≠ Current State → Take corrective action.

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

==========================================================================================================