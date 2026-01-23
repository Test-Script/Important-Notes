This lab demonstrates some of the security concerns of a default GKE cluster configuration and the corresponding hardening measures to prevent multiple paths of pod escape and cluster privilege escalation. These attack paths are relevant in the following scenarios:

An application flaw in an external facing pod that allows for Server-Side Request Forgery (SSRF) attacks.
A fully compromised container inside a pod allowing for Remote Command Execution (RCE).
A malicious internal user or an attacker with a set of compromised internal user credentials with the ability to create/update a pod in a given namespace.

Task 1. Create a simple GKE cluster

gcloud auth list

gcloud config list project

export MY_ZONE=europe-west1-d

=== Cluster Creation Command ===

gcloud container clusters create simplecluster --zone $MY_ZONE --num-nodes 2 --metadata=disable-legacy-endpoints=false

kubectl version

student-00-3f2eb73ab2e8@qwiklabs.net

KzaDKMGek9lP

qwiklabs-gcp-02-4a8b46d52e6d

Task 2. Run a Google Cloud-SDK pod

kubectl run -it --rm gcloud --image=google/cloud-sdk:latest --restart=Never -- bash

curl -s http://metadata.google.internal/computeMetadata/v1/instance/name

...snip...
Your client does not have permission to get URL <code>/computeMetadata/v1/instance/name</code> from this server. Missing Metadata-Flavor:Google header.
...snip...

curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/name

gke-simplecluster-default-pool-b57a043a-6z5v

Task 3. Deploy a pod that mounts the host filesystem

Task 4. Explore and compromise the underlying host

Task 5. Deploy a second node pool

Task 6. Run a Google Cloud-SDK pod

Task 7. Enforce Pod Security Standards

Task 8. Deploy a blocked pod that mounts the host filesystem

student_00_3f2eb73ab2e8@cloudshell:~ (qwiklabs-gcp-02-4a8b46d52e6d)$ gcloud iam service-accounts keys create key.json --iam-account "demo-developer@${MYPROJECT}.iam.gserviceaccount.com"
created key [bea5d31405ec610cec659d5509cdd8d076dcedf9] of type [json] as [key.json] for [demo-developer@qwiklabs-gcp-02-4a8b46d52e6d.iam.gserviceaccount.com]
