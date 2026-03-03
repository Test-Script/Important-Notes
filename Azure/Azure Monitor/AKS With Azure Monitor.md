==========================================================================================================
                            How Azure Monitor Collects Metrics & Logs from AKS
==========================================================================================================

Architecturally and Operationally : Azure Kubernetes Service (AKS).

# 🔎 How Azure Monitor Collects Metrics & Logs from AKS

When you enable **Container Insights** for AKS, Azure deploys a **monitoring data plane inside your cluster**.

## The full pipeline looks like this:

AKS Nodes / Pods
        ↓
Azure Monitor Agent (AMA) + Container Insights extension
        ↓
Data Collection Rule (DCR)
        ↓
Azure Monitor Pipeline
        ↓
Log Analytics Workspace / Azure Monitor Metrics DB
        ↓
Workbooks / Alerts / Grafana / KQL

# 1️⃣ Cluster-Wide Metrics Collection

## 📊 What Metrics Are Collected?

Cluster-wide metrics include:

* Node CPU / Memory
* Pod CPU / Memory
* Node conditions
* Container restarts
* Network bytes
* Disk I/O
* Kube-state-metrics (deployment replica status, etc.)
* API server metrics
* Kubelet metrics

These are **near real-time time-series metrics**.

## ⚙️ How Metrics Are Collected Internally

### A. Metrics Sources Inside AKS

Metrics are scraped from:

* kubelet (`/metrics/cadvisor`)
* kube-state-metrics
* container runtime
* node exporter style metrics
* Kubernetes API server

### B. Agent Responsible

Azure uses:

## 🔹 Azure Monitor

## 🔹 Azure Kubernetes Service

Inside AKS, Azure deploys:

* **Azure Monitor Agent (AMA)**
* Running as **DaemonSet**
* On every node

### C. Architecture View

![Image](https://learn.microsoft.com/en-us/azure/aks/media/monitor-aks/aks-monitor-data-v2.png)

![Image](https://learn.microsoft.com/en-us/azure/architecture/aws-professional/eks-to-aks/media/monitor-containers-architecture.svg)

![Image](https://kodekloud.com/kk-media/image/upload/v1752869502/notes-assets/images/Azure-Kubernetes-Service-Container-insights-for-AKS/container-insights-diagram-azure-kubernetes.jpg)

![Image](https://kodekloud.com/kk-media/image/upload/v1752869506/notes-assets/images/Azure-Kubernetes-Service-Container-insights-for-AKS/prometheus-azure-monitor-integration-diagram.jpg)

The Azure Monitor Agent:

* Scrapes node-level metrics
* Scrapes kubelet metrics
* Scrapes container metrics
* Sends data using **Data Collection Rule (DCR)**

### D. Where Metrics Are Stored?

Two places:

### 1. Azure Monitor Metrics Database

* High-performance time-series DB
* Used for near real-time alerts
* Used by autoscaling (HPA via Prometheus integration)

### 2. Log Analytics Workspace

* Stores metrics in tables like:

  * `InsightsMetrics`
  * `KubePodInventory`
  * `KubeNodeInventory`

# 2️⃣ Cluster-Wide Logs Collection

Now let’s move to logs.

## 📁 What Logs Are Collected?

### Control Plane Logs (If Enabled)

* kube-apiserver
* kube-controller-manager
* kube-scheduler
* cloud-controller-manager

### Node Logs

* kubelet
* container runtime logs

### Container Logs

* stdout / stderr of pods

### Kubernetes Events

* CrashLoopBackOff
* OOMKilled
* Scaling events
* Pod scheduling failures

## 🔹 How Logs Are Collected

### Step 1: Log Collection Agent

The **Azure Monitor Agent (AMA)**:

* Runs as DaemonSet
* Reads:

  ```
  /var/log/containers
  /var/log/pods
  journald
  kubelet logs
  ```

### Step 2: Data Collection Rule (DCR)

DCR defines:

* What logs to collect
* Namespace filtering
* Sampling rules
* Destination workspace

### Step 3: Log Ingestion

Logs are sent securely via:

* HTTPS
* Managed Identity authentication
* Azure Monitor ingestion endpoint

Stored in:

## 🔹 Azure Log Analytics

Tables include:

* `ContainerLogV2`
* `KubeEvents`
* `KubePodInventory`
* `KubeNodeInventory`
* `Perf`
* `Heartbeat`

# 3️⃣ How It Works at Kubernetes Level

Inside the cluster:

Pod Logs → Container Runtime → Node Filesystem
      ↓
Azure Monitor Agent (DaemonSet)
      ↓
DCR Processing
      ↓
Azure Monitor Backend

Important:

* No sidecar required
* No code change required
* Works cluster-wide
* Scales automatically with node scale-out

# 4️⃣ Control Plane Logs (Managed AKS Specific)

In AKS:

Control plane is managed by Microsoft.

If you enable **Diagnostic Settings** on AKS:

It sends logs from:

* API Server
* Scheduler
* Controller Manager

To:

* Log Analytics
* Event Hub
* Storage Account

This is outside the node — handled at Azure Resource layer.

# 5️⃣ Metrics vs Logs – Technical Difference

| Aspect     | Metrics             | Logs              |
| Format     | Time-series numeric | Text / structured |
| Storage    | Metrics DB          | Log Analytics     |
| Latency    | Near real-time      | Slightly higher   |
| Query      | Metrics Explorer    | KQL               |
| Alert Type | Metric Alerts       | Log Alerts        |

# 6️⃣ How It Achieves Cluster-Wide Visibility

Because:

* Agent runs as **DaemonSet**
* Collects from all nodes
* Scrapes kube-state-metrics
* Scrapes kubelet
* Reads all container logs

It does not depend on namespace — unless filtered via DCR.

# 7️⃣ Advanced Integration (For Your Level)

Since you're working with DevOps + Zero Trust patterns:

You can also:

* Enable Managed Prometheus in Azure Monitor
* Integrate with Azure Managed Grafana
* Send logs to Microsoft Sentinel
* Use Azure Policy to enforce monitoring

# 8️⃣ Security Model

Authentication used:

* Managed Identity (AKS cluster identity)
* RBAC
* DCR-based scoping
* TLS encrypted ingestion

No static credentials required.

# 9️⃣ What Happens If Agent Fails?

* Heartbeat table detects missing nodes
* Alerts can be configured
* Node restart redeploys DaemonSet
* It is self-healing

# 🔟 Real Production Recommendation (Azure Architect Level)

For enterprise AKS:

✔ Enable Container Insights
✔ Enable Control Plane Diagnostics
✔ Use DCR to filter namespaces
✔ Use Managed Prometheus for custom metrics
✔ Send Security Logs to Sentinel
✔ Set metric alerts for:

* Node NotReady
* Pod CrashLoopBackOff
* High CPU

# 🔥 Final Conceptual Understanding

Think of it like this:

Azure Monitor for AKS =

* In-cluster scraping (DaemonSet agent)
* Azure control-plane diagnostics
* Data pipeline via DCR
* Dual storage (Metrics DB + Log Analytics)
* Visualization + Alerting layer
==========================================================================================================