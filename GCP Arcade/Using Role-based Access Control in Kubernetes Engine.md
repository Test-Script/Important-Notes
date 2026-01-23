student-03-10a3d2386dff@qwiklabs.net

MtvLVGbU3hPa


gcloud auth list   ----> You can list the active account name with this command:

gcloud config list project

gcloud config set compute/region europe-west1
gcloud config set compute/zone europe-west1-c

gcloud iam service-accounts list

EMAIL: gke-tutorial-admin-rbac@qwiklabs-gcp-02-2a37a217072b.iam.gserviceaccount.com
DISABLED: False

DISPLAY NAME: Compute Engine default service account
EMAIL: 125190809027-compute@developer.gserviceaccount.com
DISABLED: False

DISPLAY NAME: GKE Tutorial Auditor RBAC
EMAIL: gke-tutorial-auditor-rbac@qwiklabs-gcp-02-2a37a217072b.iam.gserviceaccount.com
DISABLED: False

DISPLAY NAME: GKE Tutorial Owner RBAC
EMAIL: gke-tutorial-owner-rbac@qwiklabs-gcp-02-2a37a217072b.iam.gserviceaccount.com
DISABLED: False

DISPLAY NAME: Qwiklabs User Service Account
EMAIL: qwiklabs-gcp-02-2a37a217072b@qwiklabs-gcp-02-2a37a217072b.iam.gserviceaccount.com
DISABLED: False

=================================================================================================================

gcloud compute instances list

student_03_10a3d2386dff@cloudshell:~ (qwiklabs-gcp-02-2a37a217072b)$ gcloud compute instances list
NAME: gke-rbac-demo-cluster-default-pool-7413710a-h1df
ZONE: europe-west1-c
MACHINE_TYPE: n1-standard-1
PREEMPTIBLE: 
INTERNAL_IP: 10.0.96.5
EXTERNAL_IP: 
STATUS: RUNNING

NAME: gke-rbac-demo-cluster-default-pool-7413710a-nt8z
ZONE: europe-west1-c
MACHINE_TYPE: n1-standard-1
PREEMPTIBLE: 
INTERNAL_IP: 10.0.96.6
EXTERNAL_IP: 
STATUS: RUNNING

NAME: gke-tutorial-admin
ZONE: europe-west1-c
MACHINE_TYPE: f1-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.0.96.3
EXTERNAL_IP: 34.140.220.57
STATUS: RUNNING

NAME: gke-tutorial-auditor
ZONE: europe-west1-c
MACHINE_TYPE: f1-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.0.96.2
EXTERNAL_IP: 34.77.51.60
STATUS: RUNNING

NAME: gke-tutorial-owner
ZONE: europe-west1-c
MACHINE_TYPE: f1-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.0.96.4
EXTERNAL_IP: 34.76.84.126
STATUS: RUNNING

============================== Creating the RBAC rules

Create the Namespaces, Roles, and RoleBindings by logging into the admin instance and applying the rbac.yaml manifest.

gcloud compute ssh gke-tutorial-admin  ---> Login to Admin Cluster Node

============================== Authentication Plugin

sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin

echo "export USE_GKE_GCLOUD_AUTH_PLUGIN=True" >> ~/.bashrc

============================== Kubectl configuration

student-03-10a3d2386dff@gke-tutorial-admin:~$ gcloud container clusters get-credentials rbac-demo-cluster --zone 
europe-west1-c

Fetching cluster endpoint and auth data.
kubeconfig entry generated for rbac-demo-cluster.

============================== RBAC Manifest

student-03-10a3d2386dff@gke-tutorial-admin:~/manifests$ kubectl apply -f rbac.yaml 
namespace/dev created
namespace/prod created
namespace/test created
role.rbac.authorization.k8s.io/dev-ro created
clusterrole.rbac.authorization.k8s.io/all-rw created
clusterrolebinding.rbac.authorization.k8s.io/owner-binding created
rolebinding.rbac.authorization.k8s.io/auditor-binding created

----

student-03-10a3d2386dff@gke-tutorial-admin:~/manifests$ kubectl apply -f rbac.yaml 
namespace/dev created
namespace/prod created
namespace/test created
role.rbac.authorization.k8s.io/dev-ro created
clusterrole.rbac.authorization.k8s.io/all-rw created
clusterrolebinding.rbac.authorization.k8s.io/owner-binding created
rolebinding.rbac.authorization.k8s.io/auditor-binding created
student-03-10a3d2386dff@gke-tutorial-admin:~/manifests$ cat rbac.yaml 
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the License);
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an AS IS BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# This manifest defines the RBAC resources (Roles, ServiceAccounts, and Bindings)
# used in the scenario 1 of this tutorial (see the README.md).

###################################################################################
# Role Definitions
# The following roles define two sets of permissions, read-write and read-only,
# for common resources in two namespaces: dev and prod.
###################################################################################
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---

apiVersion: v1
kind: Namespace
metadata:
  name: prod

---
apiVersion: v1
kind: Namespace
metadata:
  name: test

---
# RBAC Documentation: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
# Grants read only permissions to common resource types in the dev namespace
# Because we're restricting permissions to a namespace.
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  # The namespace in which this role applies
  namespace: dev
  name: dev-ro
rules:
  # The api groups that contain the resources we want to manage
- apiGroups: ["", apps, extensions]
  # The resources to which this role grants permissions
  resources: [pods, pods/log, services, deployments, configmaps]
  # The permissions granted by this role
  verbs: [get, list, watch]

---
# Grants read-write permissions to common resource types in all namespaces
# We use a ClusterRole because we're defining cluster-wide permissions.
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  # The namespace in which this role applies
  name: all-rw
rules:
  # The api groups that contain the resources we want to manage
- apiGroups: ["", apps, extensions]
  # The resources to which this role grants permissions
  resources: [pods, services, deployments, configmaps]
  # The permissions granted by this role
  verbs: [get, list, create, update, patch, delete]

---

# Allows anyone in the manager group to read resources in any namespace.
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: owner-binding
subjects:
- kind: User
  name: gke-tutorial-owner-rbac@qwiklabs-gcp-02-2a37a217072b.iam.gserviceaccount.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: all-rw
  apiGroup: rbac.authorization.k8s.io

---
# This role binding allows anyone in the developer group to have read access
# to resources in the dev namespace.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: dev
  name: auditor-binding
subjects:
- kind: User
  name: gke-tutorial-auditor-rbac@qwiklabs-gcp-02-2a37a217072b.iam.gserviceaccount.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-ro
  apiGroup: rbac.authorization.k8s.io
  
---

==================== Getting Access of Cluster-Owner Node
student-03-10a3d2386dff@gke-tutorial-owner:~$ sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
echo "export USE_GKE_GCLOUD_AUTH_PLUGIN=True" >> ~/.bashrc
source ~/.bashrc
gcloud container clusters get-credentials rbac-demo-cluster --zone europe-west1-c
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  google-cloud-sdk-gke-gcloud-auth-plugin
0 upgraded, 1 newly installed, 0 to remove and 8 not upgraded.
Need to get 5018 B of archives.
After this operation, 19.5 kB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt cloud-sdk-bullseye/main all google-cloud-sdk-gke-gcloud-auth-plugin all 467.0.0-0 [5018 B]
Fetched 5018 B in 0s (40.2 kB/s)                                  
Selecting previously unselected package google-cloud-sdk-gke-gcloud-auth-plugin.
(Reading database ... 73983 files and directories currently installed.)
Preparing to unpack .../google-cloud-sdk-gke-gcloud-auth-plugin_467.0.0-0_all.deb ...
Unpacking google-cloud-sdk-gke-gcloud-auth-plugin (467.0.0-0) ...
Setting up google-cloud-sdk-gke-gcloud-auth-plugin (467.0.0-0) ...
Fetching cluster endpoint and auth data.
kubeconfig entry generated for rbac-demo-cluster.

=========================== Create Server in the Dev

student-03-10a3d2386dff@gke-tutorial-owner:~/manifests$ cat hello-server.yaml 
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# The manifest exposes a simple hello-server service and deployment with a single pod

# Makes the hello-server pod addressable within the cluster
kind: Service
apiVersion: v1
metadata:
  # Label and name the service
  labels:
    app: hello-server
  name: hello-server
spec:
  ports:
    # Listens on port 8080 and routes to targetPort 8080 on backend pods
  - port: 8080
    protocol: TCP
    targetPort: 8080

  # Load balance requests across all pods labeled with app=hello-server
  selector:
    app: hello-server

  # Disable session affinity, each request may be routed to a new pod
  sessionAffinity: None

  # Expose the service internally only
  type: ClusterIP

---

# Deploys a pod to service hello-server requests
apiVersion: apps/v1
kind: Deployment
metadata:
  # Label and name the deployment
  labels:
    app: hello-server
  name: hello-server
spec:

  # Only run a single pod
  replicas: 1

  # Control any pod labeled with app=hello
  selector:
    matchLabels:
      app: hello-server

  # Define pod properties
  template:
    # Ensure created pods are labeled with hello-server to match the deployment selector
    metadata:
      labels:
        app: hello-server
    spec:
      # This pod does not require access to the Kubernetes API server, so we prevent
      # even the default token from being mounted
      automountServiceAccountToken: false

      # Pod-level security context to define the default UID and GIDs under which to
      # run all container processes. We use 9999 for all IDs since it is unprivileged
      # and known to be unallocated on the node instances.
      securityContext:
        runAsUser: 9999
        runAsGroup: 9999
        fsGroup: 9999

      # Define container properties
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: hello-server

        # Describes the ports exposed on the service
        ports:
        - containerPort: 8080
          protocol: TCP

        # Container-level security settings
        # Note, containers are unprivileged by default
        securityContext:
          # Prevents the container from writing to its filesystem
          readOnlyRootFilesystem: true
student-03-10a3d2386dff@gke-tutorial-owner:~/manifests$ 

student-03-10a3d2386dff@gke-tutorial-owner:~$ kubectl create -n dev -f ./manifests/hello-server.yaml
service/hello-server created
deployment.apps/hello-server created

student-03-10a3d2386dff@gke-tutorial-owner:~$ kubectl describe deploy hello-server  -n dev
Name:                   hello-server
Namespace:              dev
CreationTimestamp:      Fri, 23 Jan 2026 02:43:39 +0000
Labels:                 app=hello-server
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=hello-server
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=hello-server
  Containers:
   hello-server:
    Image:         gcr.io/google-samples/hello-app:1.0
    Port:          8080/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Available     True    MinimumReplicasAvailable
  Progressing   True    NewReplicaSetAvailable
Events:         <none>
student-03-10a3d2386dff@gke-tutorial-owner:~$ 

kubectl create -n dev -f ./manifests/hello-server.yaml

kubectl create -n prod -f ./manifests/hello-server.yaml

kubectl create -n test -f ./manifests/hello-server.yaml

kubectl get pods -l app=hello-server --all-namespaces

Permission Evaluation

As the owner, you will also be able to view all pods.

tudent-03-10a3d2386dff@gke-tutorial-owner:~$ kubectl get pods -l app=hello-server --all-namespaces
NAMESPACE   NAME                            READY   STATUS    RESTARTS   AGE
dev         hello-server-77c4d4bf59-dg2x8   1/1     Running   0          2m51s
prod        hello-server-77c4d4bf59-mnnvk   1/1     Running   0          56s
test        hello-server-77c4d4bf59-xjpwk   1/1     Running   0          35s

On the "owner" instance list all hello-server pods in all namespaces by running:


Viewing resources as the auditor
Now you will open a new terminal, SSH into the auditor instance, and try to view all namespaces.

student_03_10a3d2386dff@cloudshell:~ (qwiklabs-gcp-02-2a37a217072b)$ gcloud compute ssh gke-tutorial-auditor
Did you mean zone [asia-southeast1-a] for instance: [gke-tutorial-auditor] (Y/n)?  n

No zone specified. Using zone [europe-west1-c] for instance: [gke-tutorial-auditor].
Warning: Permanently added 'compute.1885227940059223736' (ED25519) to the list of known hosts.
Linux gke-tutorial-auditor 5.10.0-37-cloud-amd64 #1 SMP Debian 5.10.247-1 (2025-12-11) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Jan 22 22:45:19 2026 from 104.198.174.62
Fetching cluster endpoint and auth data.
kubeconfig entry generated for rbac-demo-cluster.
student-03-10a3d2386dff@gke-tutorial-auditor:~$ sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
echo "export USE_GKE_GCLOUD_AUTH_PLUGIN=True" >> ~/.bashrc
source ~/.bashrc
gcloud container clusters get-credentials rbac-demo-cluster --zone europe-west1-c
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  google-cloud-sdk-gke-gcloud-auth-plugin
0 upgraded, 1 newly installed, 0 to remove and 8 not upgraded.
Need to get 5018 B of archives.
After this operation, 19.5 kB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt cloud-sdk-bullseye/main all google-cloud-sdk-gke-gcloud-auth-plugin all 467.0.0-0 [5018 B]
Fetched 5018 B in 0s (43.6 kB/s)                            
Selecting previously unselected package google-cloud-sdk-gke-gcloud-auth-plugin.
(Reading database ... 73983 files and directories currently installed.)
Preparing to unpack .../google-cloud-sdk-gke-gcloud-auth-plugin_467.0.0-0_all.deb ...
Unpacking google-cloud-sdk-gke-gcloud-auth-plugin (467.0.0-0) ...
Setting up google-cloud-sdk-gke-gcloud-auth-plugin (467.0.0-0) ...
Fetching cluster endpoint and auth data.
kubeconfig entry generated for rbac-demo-cluster.
student-03-10a3d2386dff@gke-tutorial-audit

kubectl get pods -l app=hello-server --all-namespaces

student-03-10a3d2386dff@gke-tutorial-auditor:~$ kubectl get pods -l app=hello-server --all-namespaces
Error from server (Forbidden): pods is forbidden: User "gke-tutorial-auditor-rbac@qwiklabs-gcp-02-2a37a217072b.iam.gserviceaccount.com" cannot list resource "pods" in API group "" at the cluster scope: requires one of ["container.pods.list"] permission(s).

kubectl get pods -l app=hello-server --namespace=dev

student-03-10a3d2386dff@gke-tutorial-auditor:~$ kubectl get pods -l app=hello-server --namespace=dev
NAME                            READY   STATUS    RESTARTS   AGE
hello-server-77c4d4bf59-dg2x8   1/1     Running   0          6m9s

student-03-10a3d2386dff@gke-tutorial-auditor:~$ kubectl get pods -l app=hello-server --namespace=test
Error from server (Forbidden): pods is forbidden: User "gke-tutorial-auditor-rbac@qwiklabs-gcp-02-2a37a217072b.iam.gserviceaccount.com" cannot list resource "pods" in API group "" in the namespace "test": requires one of ["container.pods.list"] permission(s).

student-03-10a3d2386dff@gke-tutorial-auditor:~$ kubectl get pods -l app=hello-server --namespace=prod
Error from server (Forbidden): pods is forbidden: User "gke-tutorial-auditor-rbac@qwiklabs-gcp-02-2a37a217072b.iam.gserviceaccount.com" cannot list resource "pods" in API group "" in the namespace "prod": requires one of ["container.pods.list"] permission(s).

Finally, verify that the auditor has read-only access by trying to create and delete a deployment in the dev namespace.

kubectl create -n dev -f manifests/hello-server.yaml

student-03-10a3d2386dff@gke-tutorial-auditor:~$ kubectl create -n dev -f manifests/hello-server.yaml
Error from server (Forbidden): error when creating "manifests/hello-server.yaml": services is forbidden: User "gke-tutorial-auditor-rbac@qwiklabs-gcp-02-2a37a217072b.iam.gserviceaccount.com" cannot create resource "services" in API group "" in the namespace "dev": requires one of ["container.services.create"] permission(s).
Error from server (Forbidden): error when creating "manifests/hello-server.yaml": deployments.apps is forbidden: User "gke-tutorial-auditor-rbac@qwiklabs-gcp-02-2a37a217072b.iam.gserviceaccount.com" cannot create resource "deployments" in API group "apps" in the namespace "dev": requires one of ["container.deployments.create"] permission(s).

student-03-10a3d2386dff@gke-tutorial-auditor:~$ kubectl delete deployment -n dev -l app=hello-server
Error from server (Forbidden): deployments.apps "hello-server" is forbidden: User "gke-tutorial-auditor-rbac@qwiklabs-gcp-02-2a37a217072b.iam.gserviceaccount.com" cannot delete resource "deployments" in API group "apps" in the namespace "dev": requires one of ["container.deployments.delete"] permission(s).

Author is only having viewing access in the dev namespace, No Other accesses he enjoys like view,create,delete, update.


gcloud compute ssh gke-tutorial-admin

kubectl get pods -l app=pod-labeler

student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl get pods -l app=pod-labeler
NAME                          READY   STATUS             RESTARTS      AGE
pod-labeler-dbf8bdf57-lj5fj   0/1     CrashLoopBackOff   2 (20s ago)   61s

student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl describe pod -l app=pod-labeler | tail -n 20
  kube-api-access-nr5sm:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  79s                default-scheduler  Successfully assigned default/pod-labeler-dbf8bdf57-lj5fj to gke-rbac-demo-cluster-default-pool-7413710a-nt8z
  Normal   Pulling    78s                kubelet            Pulling image "gcr.io/qwiklabs-resources/pod-labeler:0.1.5"
  Normal   Pulled     54s                kubelet            Successfully pulled image "gcr.io/qwiklabs-resources/pod-labeler:0.1.5" in 23.734s (23.734s including waiting). Image size: 373062569 bytes.
  Normal   Created    13s (x4 over 54s)  kubelet            Created container: pod-labeler
  Normal   Started    13s (x4 over 54s)  kubelet            Started container pod-labeler
  Normal   Pulled     13s (x3 over 51s)  kubelet            Container image "gcr.io/qwiklabs-resources/pod-labeler:0.1.5" already present on machine
  Warning  BackOff    12s (x4 over 50s)  kubelet            Back-off restarting failed container pod-labeler in pod pod-labeler-dbf8bdf57-lj5fj_default(cb9d4424-f892-4a48-ba7d-5f72591d819e)
  
  
 kubectl logs -l app=pod-labeler
 
 student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl logs -l app=pod-labeler
  File "/usr/local/lib/python3.8/site-packages/kubernetes/client/rest.py", line 237, in GET
    return self.request("GET", url,
  File "/usr/local/lib/python3.8/site-packages/kubernetes/client/rest.py", line 231, in request
    raise ApiException(http_resp=r)
kubernetes.client.rest.ApiException: (403)
Reason: Forbidden
HTTP response headers: HTTPHeaderDict({'Audit-Id': '1ccce5bf-693c-4c71-b061-eb9f3d5cc3d7', 'Cache-Control': 'no-cache, private', 'Content-Type': 'application/json', 'X-Content-Type-Options': 'nosniff', 'X-Kubernetes-Pf-Flowschema-Uid': 'da7027b8-c74b-409c-91ad-23712e1e07cd', 'X-Kubernetes-Pf-Prioritylevel-Uid': '3a553ec4-946f-43d3-9f4c-4772a0790d95', 'Date': 'Fri, 23 Jan 2026 02:56:13 GMT', 'Content-Length': '282'})
HTTP response body: {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"pods is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"","reason":"Forbidden","details":{"kind":"pods"},"code":403}

============= Error Fixing ========

kubectl get pod -oyaml -l app=pod-labeler

student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl get pod -oyaml -l app=pod-labeler
apiVersion: v1
items:
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      cni.projectcalico.org/containerID: 212bba13aaabf6d6884f09116ac8ccb8b8b76868d6a2286d8d396561b9a9c204
      cni.projectcalico.org/podIP: 10.0.92.136/32
      cni.projectcalico.org/podIPs: 10.0.92.136/32
    creationTimestamp: "2026-01-23T02:54:25Z"
    generateName: pod-labeler-dbf8bdf57-
    generation: 1
    labels:
      app: pod-labeler
      pod-template-hash: dbf8bdf57
    name: pod-labeler-dbf8bdf57-lj5fj
    namespace: default
    ownerReferences:
    - apiVersion: apps/v1
      blockOwnerDeletion: true
      controller: true
      kind: ReplicaSet
      name: pod-labeler-dbf8bdf57
      uid: ae09114d-63dc-4fa9-a697-80105812712e
    resourceVersion: "1769136988428063009"
    uid: cb9d4424-f892-4a48-ba7d-5f72591d819e
  spec:
    containers:
    - image: gcr.io/qwiklabs-resources/pod-labeler:0.1.5
      imagePullPolicy: IfNotPresent
      name: pod-labeler
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-nr5sm
        readOnly: true
    dnsPolicy: ClusterFirst
    enableServiceLinks: true
    nodeName: gke-rbac-demo-cluster-default-pool-7413710a-nt8z
    preemptionPolicy: PreemptLowerPriority
    priority: 0
    restartPolicy: Always
    schedulerName: default-scheduler
    securityContext:
      fsGroup: 9999
      runAsGroup: 9999
      runAsUser: 9999
    serviceAccount: default
    serviceAccountName: default
    terminationGracePeriodSeconds: 30
    tolerations:
    - effect: NoExecute
      key: node.kubernetes.io/not-ready
      operator: Exists
      tolerationSeconds: 300
    - effect: NoExecute
      key: node.kubernetes.io/unreachable
      operator: Exists
      tolerationSeconds: 300
    volumes:
    - name: kube-api-access-nr5sm
      projected:
        defaultMode: 420
        sources:
        - serviceAccountToken:
            expirationSeconds: 3607
            path: token
        - configMap:
            items:
            - key: ca.crt
              path: ca.crt
            name: kube-root-ca.crt
        - downwardAPI:
            items:
            - fieldRef:
                apiVersion: v1
                fieldPath: metadata.namespace
              path: namespace
  status:
    conditions:
    - lastProbeTime: null
      lastTransitionTime: "2026-01-23T02:54:51Z"
      status: "True"
      type: PodReadyToStartContainers
    - lastProbeTime: null
      lastTransitionTime: "2026-01-23T02:54:25Z"
      status: "True"
      type: Initialized
    - lastProbeTime: null
      lastTransitionTime: "2026-01-23T02:56:13Z"
      message: 'containers with unready status: [pod-labeler]'
      reason: ContainersNotReady
      status: "False"
      type: Ready
    - lastProbeTime: null
      lastTransitionTime: "2026-01-23T02:56:13Z"
      message: 'containers with unready status: [pod-labeler]'
      reason: ContainersNotReady
      status: "False"
      type: ContainersReady
    - lastProbeTime: null
      lastTransitionTime: "2026-01-23T02:54:25Z"
      status: "True"
      type: PodScheduled
    containerStatuses:
    - containerID: containerd://271ca029121621498ae6dec6c24510c1b3a8c69e2de5b49fd536210565d93c15
      image: gcr.io/qwiklabs-resources/pod-labeler:0.1.5
      imageID: gcr.io/qwiklabs-resources/pod-labeler@sha256:9455df40b643a1cb2fb22d219fbf21c47fdeb31263688c6b76fba773630fcd71
      lastState:
        terminated:
          containerID: containerd://271ca029121621498ae6dec6c24510c1b3a8c69e2de5b49fd536210565d93c15
          exitCode: 1
          finishedAt: "2026-01-23T02:56:13Z"
          reason: Error
          startedAt: "2026-01-23T02:56:12Z"
      name: pod-labeler
      ready: false
      resources: {}
      restartCount: 4
      started: false
      state:
        waiting:
          message: back-off 1m20s restarting failed container=pod-labeler pod=pod-labeler-dbf8bdf57-lj5fj_default(cb9d4424-f892-4a48-ba7d-5f72591d819e)
          reason: CrashLoopBackOff
      user:
        linux:
          gid: 9999
          supplementalGroups:
          - 9999
          uid: 9999
      volumeMounts:
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-nr5sm
        readOnly: true
        recursiveReadOnly: Disabled
    hostIP: 10.0.96.6
    hostIPs:
    - ip: 10.0.96.6
    phase: Running
    podIP: 10.0.92.136
    podIPs:
    - ip: 10.0.92.136
    qosClass: BestEffort
    startTime: "2026-01-23T02:54:25Z"
kind: List
metadata:
  resourceVersion: ""
student-03-10a3d2386dff@gke-tutorial-admin:~$ 

restartPolicy: Always
    schedulerName: default-scheduler
    securityContext:
      fsGroup: 9999
      runAsGroup: 9999
      runAsUser: 9999
    serviceAccount: default
    serviceAccountName: default
	

student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl apply -f manifests/pod-labeler-fix-1.yaml
role.rbac.authorization.k8s.io/pod-labeler unchanged
serviceaccount/pod-labeler unchanged
rolebinding.rbac.authorization.k8s.io/pod-labeler unchanged
deployment.apps/pod-labeler configured
student-03-10a3d2386dff@gke-tutorial-admin:~$ cat manifests/pod-labeler-fix-1.yaml 
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# This manifest deploys a sample application that accesses the Kubernetes API
# It also create a Role RoleBinding and ServiceAccount so you can manage
# API access with RBAC. The initial configuration is intentionally incomplete,
# so that it serves an example for troubleshooting (see the project README).

# Create a custom role in the default namespace that grants access to
# list pods
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pod-labeler
  namespace: default
rules:
- apiGroups: [""] # "" refers to the core API group
  resources: ["pods"]
  verbs: ["list"] # "patch" is intentionally omitted for troubleshooting (see README)

---
# Create a ServiceAccount that will be bound to the role above
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pod-labeler
  namespace: default

---
# Binds the pod-labeler ServiceAccount to the pod-labeler Role
# Any pod using the pod-labeler ServiceAccount will be granted
# API permissions based on the pod-labeler role.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pod-labeler
  namespace: default
subjects:
  # List of service accounts to bind
- kind: ServiceAccount
  name: pod-labeler
roleRef:
  # The role to bind
  kind: Role
  name: pod-labeler
  apiGroup: rbac.authorization.k8s.io

---
# Deploys a single pod to run the pod-labeler code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-labeler
  namespace: default
spec:
  replicas: 1

  # Control any pod labeled with app=pod-labeler
  selector:
    matchLabels:
      app: pod-labeler

  template:
    # Ensure created pods are labeled with app=pod-labeler to match the deployment selector
    metadata:
      labels:
        app: pod-labeler

    spec:
      # Fix 1, set the serviceAccount so RBAC rules apply
      serviceAccount: pod-labeler

      # Pod-level security context to define the default UID and GIDs under which to
      # run all container processes. We use 9999 for all IDs since it is unprivileged
      # and known to be unallocated on the node instances.
      securityContext:
        runAsUser: 9999
        runAsGroup: 9999
        fsGroup: 9999

      containers:
      - image: gcr.io/qwiklabs-resources/pod-labeler:0.1.5
        name: pod-labeler
		

student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl get deployment pod-labeler -oyaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "2"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"pod-labeler","namespace":"default"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"pod-labeler"}},"template":{"metadata":{"labels":{"app":"pod-labeler"}},"spec":{"containers":[{"image":"gcr.io/qwiklabs-resources/pod-labeler:0.1.5","name":"pod-labeler"}],"securityContext":{"fsGroup":9999,"runAsGroup":9999,"runAsUser":9999},"serviceAccount":"pod-labeler"}}}}
  creationTimestamp: "2026-01-23T02:54:25Z"
  generation: 2
  name: pod-labeler
  namespace: default
  resourceVersion: "1769137216477391001"
  uid: 87da4164-ce9c-4cbd-af13-d36ccf555af5
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: pod-labeler
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: pod-labeler
    spec:
      containers:
      - image: gcr.io/qwiklabs-resources/pod-labeler:0.1.5
        imagePullPolicy: IfNotPresent
        name: pod-labeler
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        fsGroup: 9999
        runAsGroup: 9999
        runAsUser: 9999
      serviceAccount: pod-labeler
      serviceAccountName: pod-labeler
      terminationGracePeriodSeconds: 30
status:
  conditions:
  - lastTransitionTime: "2026-01-23T02:54:25Z"
    lastUpdateTime: "2026-01-23T02:59:31Z"
    message: ReplicaSet "pod-labeler-779c866bf" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  - lastTransitionTime: "2026-01-23T03:00:16Z"
    lastUpdateTime: "2026-01-23T03:00:16Z"
    message: Deployment does not have minimum availability.
    reason: MinimumReplicasUnavailable
    status: "False"
    type: Available
  observedGeneration: 2
  replicas: 1
  unavailableReplicas: 1
  updatedReplicas: 1

student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl get deployment pod-labeler -oyaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "2"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"pod-labeler","namespace":"default"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"pod-labeler"}},"template":{"metadata":{"labels":{"app":"pod-labeler"}},"spec":{"containers":[{"image":"gcr.io/qwiklabs-resources/pod-labeler:0.1.5","name":"pod-labeler"}],"securityContext":{"fsGroup":9999,"runAsGroup":9999,"runAsUser":9999},"serviceAccount":"pod-labeler"}}}}
  creationTimestamp: "2026-01-23T02:54:25Z"
  generation: 2
  name: pod-labeler
  namespace: default
  resourceVersion: "1769137216477391001"
  uid: 87da4164-ce9c-4cbd-af13-d36ccf555af5
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: pod-labeler
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: pod-labeler
    spec:
      containers:
      - image: gcr.io/qwiklabs-resources/pod-labeler:0.1.5
        imagePullPolicy: IfNotPresent
        name: pod-labeler
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        fsGroup: 9999
        runAsGroup: 9999
        runAsUser: 9999
      serviceAccount: pod-labeler
      serviceAccountName: pod-labeler
      terminationGracePeriodSeconds: 30
status:
  conditions:
  - lastTransitionTime: "2026-01-23T02:54:25Z"
    lastUpdateTime: "2026-01-23T02:59:31Z"
    message: ReplicaSet "pod-labeler-779c866bf" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  - lastTransitionTime: "2026-01-23T03:00:16Z"
    lastUpdateTime: "2026-01-23T03:00:16Z"
    message: Deployment does not have minimum availability.
    reason: MinimumReplicasUnavailable
    status: "False"
    type: Available
  observedGeneration: 2
  replicas: 1
  unavailableReplicas: 1
  updatedReplicas: 1



============================= Diagnosing insufficient privileges

student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl get pods -l app=pod-labeler
NAME                          READY   STATUS             RESTARTS      AGE
pod-labeler-779c866bf-rtnrh   0/1     CrashLoopBackOff   4 (69s ago)   2m46s

tudent-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl get pods -l app=pod-labeler
NAME                          READY   STATUS             RESTARTS      AGE
pod-labeler-779c866bf-rtnrh   0/1     CrashLoopBackOff   4 (69s ago)   2m46s
student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl logs -l app=pod-labeler
  File "/usr/local/lib/python3.8/site-packages/kubernetes/client/rest.py", line 292, in PATCH
    return self.request("PATCH", url,
  File "/usr/local/lib/python3.8/site-packages/kubernetes/client/rest.py", line 231, in request
    raise ApiException(http_resp=r)
kubernetes.client.rest.ApiException: (403)
Reason: Forbidden
HTTP response headers: HTTPHeaderDict({'Audit-Id': '77f4e467-ca15-4487-a9f8-1431239aee6b', 'Cache-Control': 'no-cache, private', 'Content-Type': 'application/json', 'X-Content-Type-Options': 'nosniff', 'X-Kubernetes-Pf-Flowschema-Uid': 'da7027b8-c74b-409c-91ad-23712e1e07cd', 'X-Kubernetes-Pf-Prioritylevel-Uid': '3a553ec4-946f-43d3-9f4c-4772a0790d95', 'Date': 'Fri, 23 Jan 2026 03:02:34 GMT', 'Content-Length': '356'})
HTTP response body: {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"pods \"pod-labeler-779c866bf-rtnrh\" is forbidden: User \"system:serviceaccount:default:pod-labeler\" cannot patch resource \"pods\" in API group \"\" in the namespace \"default\"","reason":"Forbidden","details":{"name":"pod-labeler-779c866bf-rtnrh","kind":"pods"},"code":403}

student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl get rolebinding pod-labeler -oyaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"rbac.authorization.k8s.io/v1","kind":"RoleBinding","metadata":{"annotations":{},"name":"pod-labeler","namespace":"default"},"roleRef":{"apiGroup":"rbac.authorization.k8s.io","kind":"Role","name":"pod-labeler"},"subjects":[{"kind":"ServiceAccount","name":"pod-labeler"}]}
  creationTimestamp: "2026-01-23T02:54:25Z"
  name: pod-labeler
  namespace: default
  resourceVersion: "1769136865345935023"
  uid: c11d0f1b-a74e-4ff8-83f8-1ca841874b7b
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-labeler
subjects:
- kind: ServiceAccount
  name: pod-labeler
student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl get role pod-labeler -oyaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"rbac.authorization.k8s.io/v1","kind":"Role","metadata":{"annotations":{},"name":"pod-labeler","namespace":"default"},"rules":[{"apiGroups":[""],"resources":["pods"],"verbs":["list"]}]}
  creationTimestamp: "2026-01-23T02:54:25Z"
  name: pod-labeler
  namespace: default
  resourceVersion: "1769136865213375019"
  uid: 876f9546-5098-488d-b73b-543b2ad34f32
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - list
  
  
  student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl delete pod -l app=pod-labeler
pod "pod-labeler-779c866bf-rtnrh" deleted


student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl get pods --show-labels
NAME                          READY   STATUS    RESTARTS   AGE   LABELS
pod-labeler-779c866bf-ncgdw   1/1     Running   0          23s   app=pod-labeler,pod-template-hash=779c866bf,updated=1769137827.9265292

student-03-10a3d2386dff@gke-tutorial-admin:~$ kubectl logs -l app=pod-labeler
Attempting to list pods
labeling pod pod-labeler-779c866bf-ncgdw
Attempting to list pods
labeling pod pod-labeler-779c866bf-ncgdw
Attempting to list pods
labeling pod pod-labeler-779c866bf-ncgdw

Key take-aways
Container and API server logs will be your best source of clues for diagnosing RBAC issues.
Use RoleBindings or ClusterRoleBindings to determine which role is specifying the permissions for a pod.
API server logs can be found in Stackdriver under the Kubernetes resource.
Not all API calls will be logged to Stackdriver. Frequent, or verbose payloads are omitted by the Kubernetes' audit policy used in Kubernetes Engine. The exact policy will vary by Kubernetes version, but can be found in the open source codebase.