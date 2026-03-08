=# ArgoCD Installation Guide

ArgoCD is a powerful GitOps tool for Kubernetes that helps manage applications declaratively. This guide covers different installation methods, with a focus on practical setups for local development and production environments.

## Prerequisites

Before installing ArgoCD, ensure you have:
- A running Kubernetes cluster (e.g., Minikube, AKS, EKS)
- `kubectl` configured and connected to your cluster
- `helm` installed if using the Helm method
- Sufficient cluster resources (at least 1 CPU and 1GB RAM recommended)

In my experience, starting with a local setup like Minikube is great for learning, but for production, always use proper security configurations.

## Installation Methods

### Method 1: Using Kubernetes Manifests (Recommended for Quick Setup)

This method uses the official YAML manifests directly from the ArgoCD repository. It's straightforward and gives you full control.

1. Create the ArgoCD namespace:
   ```bash
   kubectl create namespace argocd
   ```

2. Apply the installation manifest:
   ```bash
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. Wait for pods to be ready:
   ```bash
   kubectl get pods -n argocd
   ```
   You should see pods like `argocd-server`, `argocd-repo-server`, etc., in Running state.

4. Access the UI locally:
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```
   Then open http://localhost:8080 in your browser.

5. Get the initial admin password:
   ```bash
   kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
   ```
   **Important:** Change this password immediately after first login for security.

### Method 2: Using Helm Charts (Better for Production)

Helm provides more customization options and easier upgrades. I prefer this for production deployments.

1. Add the ArgoCD Helm repository:
   ```bash
   helm repo add argo https://argoproj.github.io/argo-helm
   helm repo update
   ```

2. Install ArgoCD:
   ```bash
   helm install argocd argo/argo-cd --namespace argocd --create-namespace
   ```

3. Access the UI:
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```

4. Get the admin password (same as above).

After installation, you'll see output like:
```
NAME: argocd
LAST DEPLOYED: [Date]
NAMESPACE: argocd
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
In order to access the server UI you have the following options:

1. kubectl port-forward service/argocd-server -n argocd 8080:443
   and then open the browser on http://localhost:8080 and accept the certificate

2. enable ingress in the values file `server.ingress.enabled` and either
   - Add the annotation for ssl passthrough
   - Set the `configs.params."server.insecure"` in the values file and terminate SSL at your ingress

After reaching the UI the first time you can login with username: admin and the random password generated during the installation.
```

## Installation in Minikube (Local Development Setup)

For local development with Minikube, especially in a VM environment, follow these steps for proper connectivity.

### 1. Prepare Minikube and ArgoCD

- Install ArgoCD using manifests as above.
- Enable insecure mode for easier local access:
  ```bash
  kubectl patch configmap argocd-cmd-params-cm -n argocd --type merge -p '{"data": {"server.insecure": "true"}}'
  ```
- Restart the server:
  ```bash
  kubectl rollout restart deployment argocd-server -n argocd
  ```

### 2. Identify Network Connectivity

Find the correct IP for port forwarding. In a VM setup:
- Run `ip addr show` to list interfaces.
- Look for the host-only adapter (often `enp0s8`) with an IP like `192.168.56.103`.

### 3. Expose the Service

Port forward to the VM's network interface:
```bash
sudo KUBECONFIG=$HOME/.kube/config kubectl port-forward --address 192.168.56.103 -n argocd svc/argocd-server 8443:8080
```
- `8443`: External port on your laptop
- `8080`: Internal ArgoCD port (insecure mode)

### 4. Configure Host Laptop

Add an entry to `/etc/hosts` on your laptop:
```
192.168.56.103 argocd.local
```
Then access via https://argocd.local:8443.

## Post-Installation Steps

1. **Change Default Password:** Always update the admin password.
2. **Configure RBAC:** Set up proper access controls.
3. **Set Up Repositories:** Connect your Git repositories.
4. **Deploy First App:** Try syncing a simple application to verify everything works.

## Troubleshooting

- **Pods not starting:** Check cluster resources and logs with `kubectl logs -n argocd <pod-name>`.
- **UI not accessible:** Verify port forwarding and firewall settings.
- **Certificate issues:** Use insecure mode for local dev, or configure proper TLS for production.

Remember, ArgoCD is powerful but has a learning curve. Start simple and gradually add complexity as you get comfortable.

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
