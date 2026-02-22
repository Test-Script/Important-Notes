==========================================================================================================
                                        ArgoCD Installation
==========================================================================================================

Method 1: Using Kubernetes Manifest Files

    kubectl create namespace argocd

    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

    kubectl get pods -n argocd

    kubectl port-forward svc/argocd-server -n argocd 8080:443

    kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d

Method 2: Using Helm Charts

    helm repo add argo https://argoproj.github.io/argo-helm

    helm repo update

    helm install argocd argo/argo-cd --namespace argocd --create-namespace

    kubectl port-forward svc/argocd-server -n argocd 8080:443

==========================================================================================================
                            Result After Installation Of ArgoCD
==========================================================================================================

vboxuser@admin:~$ helm install argocd argo/argo-cd --namespace argocd --create-namespace
I0222 06:37:37.503519   15777 warnings.go:110] "Warning: unrecognized format \"int64\""
I0222 06:37:37.702399   15777 warnings.go:110] "Warning: unrecognized format \"int64\""
I0222 06:37:38.446846   15777 warnings.go:110] "Warning: unrecognized format \"int64\""
NAME: argocd
LAST DEPLOYED: Sun Feb 22 06:37:27 2026
NAMESPACE: argocd
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
In order to access the server UI you have the following options:

1. kubectl port-forward service/argocd-server -n argocd 8080:443

    and then open the browser on http://localhost:8080 and accept the certificate

2. enable ingress in the values file `server.ingress.enabled` and either
      - Add the annotation for ssl passthrough: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-1-ssl-passthrough
      - Set the `configs.params."server.insecure"` in the values file and terminate SSL at your ingress: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-2-multiple-ingress-objects-and-hosts


After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by running:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

(You should delete the initial secret afterwards as suggested by the Getting Started Guide: https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)

==========================================================================================================
                                Installation Of ArgoCD In Minikube With Manifest
=========================================================================================================
---

### 1. Prepare Minikube and ArgoCD

Start by ensuring the environment is set up to handle external traffic.

* **Install ArgoCD:** `kubectl create namespace argocd`
`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`
* **Set Insecure Mode:** (Crucial for easy local access)
`kubectl patch configmap argocd-cmd-params-cm -n argocd --type merge -p '{"data": {"server.insecure": "true"}}'`
* **Restart Server:** `kubectl rollout restart deployment argocd-server -n argocd`

---

### 2. Identify Connectivity (The "Bridge")

Find the IP address that your physical laptop can actually "talk" to.

* **Check VM IPs:** Run `ip addr show`.
* **Identify Host-Only IP:** Look for the interface (often `enp0s8`) with an IP like `192.168.56.103`.

---

### 3. Expose the Service

Use `kubectl port-forward` to bridge the internal Kubernetes network to the VM’s network interface.

```bash
# Point to your specific Host-Only IP
sudo KUBECONFIG=$HOME/.kube/config kubectl port-forward --address 192.168.56.103 -n argocd svc/argocd-server 8443:8080

```

* **8443:** The port you use on your laptop browser.
* **8080:** The internal port ArgoCD listens on (when in insecure mode).

---

### 4. Configure the Host Laptop

Your physical laptop needs to know where to send traffic for the `argocd.local` domain.

* **Edit Hosts File:** Add `192.168.56.103 argocd.local` to `C:\Windows\System32\drivers\etc\hosts` (Windows) or `/etc/hosts` (Mac/Linux).
* **Access the UI:**
Open an **Incognito** window and go to: `http://argocd.local:8443`

---

### 5. Authentication

Get your credentials to log in for the first time.

* **Username:** `admin`
* **Get Password:**
`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode; echo`

---

### Summary of Component roles

| Component | Responsibility |
| --- | --- |
| **VirtualBox Host-Only Adapter** | Provides the physical network path from laptop to VM. |
| **Insecure Mode** | Disables backend SSL so you don't fight certificate errors locally. |
| **Port Forward (`--address`)** | Listens on the VM's external IP instead of just `localhost`. |
| **Hosts File** | Acts as a local DNS so you can use pretty URLs. |

==========================================================================================================
                                Installation Of ArgoCD In Minikube With Heml
==========================================================================================================

1. Add the Argo Helm Repository

First, you need to tell Helm where to find the ArgoCD charts.

    helm repo add argo https://argoproj.github.io/argo-helm
    helm repo update

2. Create a values.yaml for Minikube

    Instead of running manual kubectl patch commands, we define our requirements in a file. Create a file named argocd-values.yaml:

===========

server:
  # This replaces the manual 'insecure' patch we did earlier
  extraArgs:
    - --insecure

  ingress:
    enabled: true
    ingressClassName: nginx
    hostname: argocd.local
    annotations:
      nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
      nginx.ingress.kubernetes.io/ssl-redirect: "false"

===========

3. Install via Helm

Now, install the chart using your custom values.

helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  -f argocd-values.yaml

4. Bridge to your Laptop (The Step You Found Success With)

Even with Helm, because you are in a VirtualBox VM, you still need to open the "tunnel" so your laptop can see the service.

Run the port-forward:

# Using the Host-Only IP we identified earlier
sudo KUBECONFIG=$HOME/.kube/config kubectl port-forward --address 192.168.56.103 -n argocd svc/argocd-server 8443:80

(Note: Helm maps the service to port 80 when in insecure mode, so we forward 8443 -> 80).

http://argocd.local:8443/login?return_url=http%3A%2F%2Fargocd.local%3A8443%2Fapplications

Gemini said
That’s great to hear! It’s a common hurdle when working with Kubernetes in a virtualized environment. Since you’re building your skills for a move into the Cloud/DevOps domain, documenting this workflow is a solid practice.

Here is the summarized, "golden path" for running ArgoCD in Minikube on a VirtualBox VM and accessing it from your laptop.

1. Prepare Minikube and ArgoCD
Start by ensuring the environment is set up to handle external traffic.

Install ArgoCD: kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Set Insecure Mode: (Crucial for easy local access)
kubectl patch configmap argocd-cmd-params-cm -n argocd --type merge -p '{"data": {"server.insecure": "true"}}'

Restart Server: kubectl rollout restart deployment argocd-server -n argocd

2. Identify Connectivity (The "Bridge")
Find the IP address that your physical laptop can actually "talk" to.

Check VM IPs: Run ip addr show.

Identify Host-Only IP: Look for the interface (often enp0s8) with an IP like 192.168.56.103.

3. Expose the Service
Use kubectl port-forward to bridge the internal Kubernetes network to the VM’s network interface.

Bash
# Point to your specific Host-Only IP
sudo KUBECONFIG=$HOME/.kube/config kubectl port-forward --address 192.168.56.103 -n argocd svc/argocd-server 8443:8080
8443: The port you use on your laptop browser.

8080: The internal port ArgoCD listens on (when in insecure mode).

4. Configure the Host Laptop
Your physical laptop needs to know where to send traffic for the argocd.local domain.

Edit Hosts File: Add 192.168.56.103 argocd.local to C:\Windows\System32\drivers\etc\hosts (Windows) or /etc/hosts (Mac/Linux).

Access the UI:
Open an Incognito window and go to: http://argocd.local:8443

5. Authentication
Get your credentials to log in for the first time.

Username: admin

Get Password:
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode; echo

==========================================================================================================
