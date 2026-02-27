## Deployment Group in Azure DevOps Framework

1️⃣ What is a Deployment Group?

Deployment Group in Azure DevOps is a logical grouping of target machines (VMs or physical servers) used for agent-based deployments.

It allows you to:

* Install a deployment agent on each target server
* Organize servers into groups (Prod, QA, UAT, etc.)
* Deploy applications directly from Azure DevOps pipelines to those machines

It is primarily used in **Classic Release Pipelines**.

---

2️⃣ Architecture Overview

![Image](https://learn.microsoft.com/en-us/azure/devops/pipelines/architectures/media/azure-pipelines-app-service-variant-architecture.svg?view=azure-devops)

![Image](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/media/agent-connections-devops.png?view=azure-devops)

![Image](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/media/what-is-release-management/understand-rm-05.png?view=azure-devops)

![Image](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/media/definition-01.png?view=azure-devops)

## WorkFlow:

Azure DevOps Project
        ↓
Release Pipeline
        ↓
Deployment Group
        ↓
Deployment Group Agent (installed on VM)
        ↓
Application deployed on target machine.

3️⃣ How It Works: 

## Step 1: Create Deployment Group

* Go to **Pipelines → Deployment Groups**
* Create a new group (e.g., `Prod-Web-Servers`)

## Step 2: Install Agent on Target Machine

Azure DevOps generates a PowerShell script:

```powershell
.\config.cmd --deploymentgroup --deploymentgroupname "Prod-Web-Servers"
```

This:

* Registers machine to Azure DevOps
* Installs self-hosted agent
* Connects machine securely

#Step 3: Use in Release Pipeline

* Add **Deployment Group Job**
* Select the created group
* Add tasks (IIS deploy, script, copy files, etc.)

---

4️⃣ Deployment Group vs Other Azure DevOps Concepts

| Feature      | Deployment Group  | Agent Pool          | Environment              |
| ------------ | ----------------- | ------------------- | ------------------------ |
| Used In      | Classic Release   | YAML & Classic      | YAML                     |
| Target       | Specific machines | Build/Deploy agents | Modern deployment target |
| Tag Support  | Yes               | No                  | Yes                      |
| Future-proof | ❌ Legacy          | ✅ Yes               | ✅ Recommended       |

---

5️⃣ Deployment Group vs Environment (Modern Approach)

Microsoft now recommends using **Environments** in YAML pipelines instead of Deployment Groups.

| Deployment Group | Environment             |
| ---------------- | ----------------------- |
| Classic only     | YAML pipelines          |
| Older model      | Modern DevOps practice  |
| Agent-based      | Supports VM, Kubernetes |

6️⃣ Real-Time Use Case Example

Suppose you have:

* 3 IIS servers in Production
* 2 API servers in UAT

You can:

```
Prod-Group
  ├── VM1 (tag: web)
  ├── VM2 (tag: web)
  ├── VM3 (tag: api)

UAT-Group
  ├── VM4
  ├── VM5
```

Then deploy using tags:

```
Only deploy to tag = web
```
---

7️⃣ When Should YOU Use It?

Based on your profile:

* You work with **AKS**
* You use **Helm**
* You use **Azure DevOps YAML pipelines**

👉 **You should avoid Deployment Groups unless maintaining legacy infrastructure.**

Prefer:

* Environments
* Multi-stage YAML pipelines
* Kubernetes Service Connections

---

8️⃣ Key Technical Characteristics

* Uses **self-hosted agent**
* Secure communication via PAT
* Supports machine tagging
* Can run PowerShell, Bash, IIS tasks
* Supports rolling deployment in Classic

---

9️⃣ Limitations

* Not fully compatible with modern YAML-only pipelines
* Considered legacy by Microsoft
* No native Kubernetes awareness
* Less flexible than Environments

---

## 🔎 Summary

Deployment Group = Agent-based machine grouping for Classic Release deployment.

It is useful for:

* VM-based deployments
* IIS deployments
* On-prem servers

But for modern DevOps (especially your Azure + AKS setup), use:

```
Environment + YAML pipeline
```