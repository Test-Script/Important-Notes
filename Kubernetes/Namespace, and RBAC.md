==========================================================================================================
                                        Namespace + RBAC
==========================================================================================================

- Create a namespace for a team.
- Allow that team to manage workloads inside it.
- Prevent access to other namespaces or cluster-wide objects.

Scenario : 

    We want a namespace called: team-a

    And a user (or service account) called: team-a-user

They should be able to:

✔ create Pods / Deployments / Services
✔ read ConfigMaps & Secrets
❌ NOT touch other namespaces
❌ NOT modify nodes or cluster-wide resources

==========================================================================================================

Step 1:

Create Namespace:

    apiVersion: v1
    kind: namespace
    metadata:
        name: team-a

Step 2:

Create a User Identity (ServiceAccount):

    In Kubernetes RBAC, permissions are bound to users, groups, or ServiceAccounts.

    We will use a ServiceAccount:

    apiVersion: v1
    kind: ServiceAccount
    metadata:
        name: team-a-sa
        namespace: team-a

===================

vboxuser@admin:~/kubernetes_Manifest$ kubectl get sa
NAME        SECRETS   AGE
default     0         15m
team-a-sa   0         12m
vboxuser@admin:~/kubernetes_Manifest$ kubectl describe sa team-a-sa
Name:                team-a-sa
Namespace:           team-a
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Events:              <none>

===================

Step 3:

Create a Role (Namespace-Scoped Permissions):

This Role allows full workload control inside team-a only.

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
        name: team-a-developer
        namespace: team-a
    rules:
    - apiGroups: [""]
        resources: ["pods", "services", "configmaps", "secrets"]
        verbs: ["get", "list", "watch", "create", "update", "delete"]

    - apiGroups: ["apps"]
        resources: ["deployments", "replicasets", "statefulsets"]
        verbs: ["get", "list", "watch", "create", "update", "delete"]


=====================

vboxuser@admin:~/kubernetes_Manifest$ kubectl get role
NAME               CREATED AT
team-a-developer   2026-02-07T17:54:35Z
vboxuser@admin:~/kubernetes_Manifest$ kubectl describe role team-a-developer
Name:         team-a-developer
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources          Non-Resource URLs  Resource Names  Verbs
  ---------          -----------------  --------------  -----
  configmaps         []                 []              [get list watch create udpate delete]
  pods               []                 []              [get list watch create udpate delete]
  secrets            []                 []              [get list watch create udpate delete]
  services           []                 []              [get list watch create udpate delete]
  deployments.apps   []                 []              [get list watch create update delete]
  replicasets.apps   []                 []              [get list watch create update delete]
  statefulsets.apps  []                 []              [get list watch create update delete]

=======================

Step 4:

Bind the Role to the ServiceAccount:

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: team-a-binding
  namespace: team-a
subjects:
  - kind: ServiceAccount
    name: team-a-sa
    namespace: team-a
roleRef:
  kind: Role
  name: team-a-developer
  apiGroup: rbac.authorization.k8s.io

====================

vboxuser@admin:~/kubernetes_Manifest$ kubectl describe rolebinding team-a-binding
Name:         team-a-binding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  team-a-developer
Subjects:
  Kind            Name       Namespace
  ----            ----       ---------
  ServiceAccount  team-a-sa  team-a

==========================================================================================================
                                    Verify Permissions : Core Testing Tool
=========================================================================================================

kubectl auth can-i ... : If I were this identity, would Kubernetes allow this action?


✅ Test 1 — Allowed: Create Pods in team-a

vboxuser@admin:~/kubernetes_Manifest$ kubectl auth can-i create pods \
  -n team-a \
  --as=system:serviceaccount:team-a:team-a-sa
yes

❌ Test 2 — Blocked: Create Pods in default namespace

vboxuser@admin:~/kubernetes_Manifest$ kubectl auth can-i create pods \
  -n default \
  --as=system:serviceaccount:team-a:team-a-sa
no

❌ Test 3 — Blocked: Read Nodes (cluster-wide)

vboxuser@admin:~/kubernetes_Manifest$ kubectl auth can-i get nodes \
  --as=system:serviceaccount:team-a:team-a-sa

Warning: resource 'nodes' is not namespace scoped

no

✅ Test 4 — Allowed: Read Secrets in team-a

vboxuser@admin:~/kubernetes_Manifest$ kubectl auth can-i get secrets \
  -n team-a \
  --as=system:serviceaccount:team-a:team-a-sa
yes

❌ Test 5 — Blocked: Delete Namespace

vboxuser@admin:~/kubernetes_Manifest$ kubectl auth can-i delete namespaces \
  --as=system:serviceaccount:team-a:team-a-sa
Warning: resource 'namespaces' is not namespace scoped

no

✅ Test 6 — Allowed: Create Deployments

vboxuser@admin:~/kubernetes_Manifest$ kubectl auth can-i create deployments.apps \
  -n team-a \
  --as=system:serviceaccount:team-a:team-a-sa
yes

❌ Test 7 — Blocked: Create CRDs

vboxuser@admin:~/kubernetes_Manifest$ kubectl auth can-i create customresourcedefinitions \
  --as=system:serviceaccount:team-a:team-a-sa
Warning: resource 'customresourcedefinitions' is not namespace scoped in group 'apiextensions.k8s.io'

no

🔎 Test 8 — List Everything This Identity Can Do

vboxuser@admin:~/kubernetes_Manifest$ kubectl auth can-i --list \
  -n team-a \
  --as=system:serviceaccount:team-a:team-a-sa
Resources                                       Non-Resource URLs                      Resource Names   Verbs
selfsubjectreviews.authentication.k8s.io        []                                     []               [create]
selfsubjectaccessreviews.authorization.k8s.io   []                                     []               [create]
selfsubjectrulesreviews.authorization.k8s.io    []                                     []               [create]
configmaps                                      []                                     []               [get list watch create udpate delete]
pods                                            []                                     []               [get list watch create udpate delete]
secrets                                         []                                     []               [get list watch create udpate delete]
services                                        []                                     []               [get list watch create udpate delete]
deployments.apps                                []                                     []               [get list watch create update delete]
replicasets.apps                                []                                     []               [get list watch create update delete]
statefulsets.apps                               []                                     []               [get list watch create update delete]
                                                [/.well-known/openid-configuration/]   []               [get]
                                                [/.well-known/openid-configuration]    []               [get]
                                                [/api/*]                               []               [get]
                                                [/api]                                 []               [get]
                                                [/apis/*]                              []               [get]
                                                [/apis]                                []               [get]
                                                [/healthz]                             []               [get]
                                                [/healthz]                             []               [get]
                                                [/livez]                               []               [get]
                                                [/livez]                               []               [get]
                                                [/openapi/*]                           []               [get]
                                                [/openapi]                             []               [get]
                                                [/openid/v1/jwks/]                     []               [get]
                                                [/openid/v1/jwks]                      []               [get]
                                                [/readyz]                              []               [get]
                                                [/readyz]                              []               [get]
                                                [/version/]                            []               [get]
                                                [/version/]                            []               [get]
                                                [/version]                             []               [get]
                                                [/version]                             []               [get]
vboxuser@admin:~/kubernetes_Manifest$

Note: This prints all verbs/resources allowed in that namespace.

Very useful for audits.



=================================
Detailed Information On the Role
=================================

Kubernetes API Groups and Resources

Format: API Group | Common Resources

"" (Core) | Pods, Services, ConfigMaps, Secrets, PersistentVolumeClaims

apps | Deployments, StatefulSets, DaemonSets

batch | Jobs, CronJobs

networking.k8s.io | Ingress, NetworkPolicies

rbac.authorization.k8s.io | Roles, RoleBindings

storage.k8s.io | StorageClasses, VolumeAttachments

autoscaling | HorizontalPodAutoscalers

policy | PodDisruptionBudgets


================================
YAML Notation for the API Groups
================================

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: full-reference-reader
rules:
  # 1. Core Group (The empty string)
  - apiGroups: [""]
    resources: ["pods", "services", "configmaps", "secrets", "nodes", "namespaces"]
    verbs: ["get", "list", "watch"]

  # 2. Workload Groups
  - apiGroups: ["apps"]
    resources: ["deployments", "statefulsets", "daemonsets", "replicasets"]
    verbs: ["get", "list", "watch"]

  # 3. Batch/Automation Groups
  - apiGroups: ["batch"]
    resources: ["jobs", "cronjobs"]
    verbs: ["get", "list", "watch"]

  # 4. Networking Groups
  - apiGroups: ["networking.k8s.io"]
    resources: ["ingresses", "networkpolicies"]
    verbs: ["get", "list", "watch"]

  # 5. RBAC Self-Management
  - apiGroups: ["rbac.authorization.k8s.io"]
    resources: ["roles", "rolebindings", "clusterroles", "clusterrolebindings"]
    verbs: ["get", "list", "watch"]

  # 6. Storage and Autoscaling
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses", "persistentvolumes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["autoscaling"]
    resources: ["horizontalpodautoscalers"]
    verbs: ["get", "list", "watch"]

  # 7. Monitoring/Metrics (If Metrics Server is installed)
  - apiGroups: ["metrics.k8s.io"]
    resources: ["pods", "nodes"]
    verbs: ["get", "list"]

  # 8. Custom Resource Definitions (CRDs)
  # Replace with the specific group of your CRD (e.g., "cert-manager.io")
  - apiGroups: ["apiextensions.k8s.io"]
    resources: ["customresourcedefinitions"]
    verbs: ["get", "list"]

==========================================================================================================