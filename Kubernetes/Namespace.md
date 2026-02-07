======================================================================================
Namespace by itself only creates a resource-scoping boundary, not a network boundary
======================================================================================

A Namespace provides logical isolation for:

    API objects (Pods, Services, ConfigMaps, Secrets, Deployments)

    RBAC permissions

    ResourceQuotas & LimitRanges

    NetworkPolicy attachment points (but not enforcement by default)

What a Namespace Does NOT Do

A Namespace alone does not:

    block Pod-to-Pod traffic

    isolate IP ranges

    prevent cross-namespace communication

    create firewalls

    enforce zero-trust networking

Any Pod can talk to any other Pod across namespaces (cluster-wide flat network).

======================================================================================
Mental Model
======================================================================================

Namespace = folder in the API
NetworkPolicy = firewall rules
CNI plugin = enforcer

======================================================================================
Network Isolation
======================================================================================

Network isolation comes from:

    NetworkPolicy objects

    A CNI plugin that supports them (Calico, Cilium, Antrea, etc.)

Without NetworkPolicy → traffic is allow-all.



==============================================================================================================================================================================

Resource limits does not embed directly inside a Namespace object.

==============================================================================================================================================================================

Namespace
   +
ResourceQuota   → caps total usage in that namespace
LimitRange      → default/min/max per Pod/container

=======================================================================================
                                Example Of Namespace
=======================================================================================

Create Namespace

    apiVersion: v1
    kind: Namespace
    metadata:
    name:team-a

================

vboxuser@admin:~/kubernetes_Manifest$ kubectl describe namespaces team-a
Name:         team-a
Labels:       kubernetes.io/metadata.name=team-a
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.

================

Add ResourceQuota (namespace-wide cap)

This prevents the namespace from consuming more than a fixed amount of cluster resources.

    apiVersion: v1
    kind: ResourceQuota
    metadata:
    name: dev-team-quota
    namespace: dev-team
    spec:
    hard:
        requests.cpu: "4"
        requests.memory: 8Gi            # All Pods combined may request ≤ 4 CPUs
        limits.cpu: "8"                 # All Pods combined may limit ≤ 8 CPUs
        limits.memory: 16Gi             # Total Memory Limits <= 16GB
        pods: "20"                      # Max Pod limit : 20

Add LimitRange (per-container defaults)

This enforces default values when developers forget to specify them.

    apiVersion: v1
    kind: LimitRange
    metadata:
    name: dev-team-limits
    namespace: dev-team
    spec:
    limits:
        - type: Container
        default:
            cpu: "500m"
            memory: 512Mi
        defaultRequest:
            cpu: "250m"
            memory: 256Mi
        min:
            cpu: "100m"
            memory: 128Mi
        max:
            cpu: "2"
            memory: 2Gi


    If no limits are set → defaults apply

    Containers can’t exceed 2 CPU / 2Gi RAM

    Containers must request at least 100m CPU

kubectl apply -f namespace.yaml

kubectl apply -f quota.yaml

kubectl apply -f limits.yaml

=======================================================================================
Verify Everything
=======================================================================================

kubectl get quota -n dev-team
kubectl get limitrange -n dev-team
kubectl describe quota dev-team-quota -n dev-team

==================
Mental Model
==================

Namespace = boundary
ResourceQuota = total budget
LimitRange = per-container guardrails