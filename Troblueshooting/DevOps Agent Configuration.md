========================== Azure DevOps Agent Configuration =========================================

Agent Configuration Path

https://dev.azure.com/tejastarghale47/

PAT Token :

Enter Agnet Pool Name : Self_Agent_Pool

Enter the Agent Name : Localhost (Default)

vstsagent.tejastarghale47.Self_Agent_Pool.TEJAS : Windows 11 Desktop

vsts.agent.tejastarghale47.Self_Agent_Pool.localhost.service : RHEL 9 System

========================================================================================================


Question : Failed to start vsts.agent.tejastarghale47.Self_Agent_Pool.SonarQube.service: Unit vsts.agent.tejastarghale47.Self_Agent_Pool.SonarQube.service not found.

Answer : 

Below is a **precise, step-by-step troubleshooting guide** in **Notepad++ format**, focused on **self-hosted Linux Azure DevOps agents**.

---

## 🔴 Problem Summary

```
Failed to start vsts.agent.tejastarghale47.Self_Agent_Pool.SonarQube.service
Unit not found
```

This error means **the systemd service file for the agent was never created or was removed**, so `svc.sh start` has nothing to start.

---

## 1️⃣ Root Cause (Very Important)

`svc.sh start` **does NOT create** the service.
It only **starts an existing systemd unit**.

Your agent is either:

* Not installed as a service
* Installed under a **different agent name**
* Service file deleted or never generated

---

## 2️⃣ Verify Agent Directory (Must Match)

Run from the **exact agent home directory**:

```
cd ~/agent
ls
```

You **must** see:

```
svc.sh
config.sh
run.sh
```

If not → you are in the **wrong directory**

---

## 3️⃣ Check Whether Service Exists in systemd

Run:

```
systemctl list-unit-files | grep vsts
```

OR:

```
ls /etc/systemd/system | grep vsts
```

### Expected:

```
vsts.agent.<org>.<pool>.<agent>.service
```

### Your output:

```
❌ Nothing found → service was never installed
```

---

## 4️⃣ Reinstall the Agent Service (Correct Fix)

From inside `~/agent`:

### Step 1: Remove old configuration (safe)

```
sudo ./svc.sh uninstall
./config.sh remove
```

If uninstall says “not found” → ignore and continue.

---

### Step 2: Reconfigure the agent

```
./config.sh
```

Answer carefully:

```
Server URL      : https://dev.azure.com/<your-org>
Authentication  : PAT
Agent Pool      : Self_Agent_Pool
Agent Name      : SonarQube
```

⚠️ **Agent name MUST match** what you expect in the service name.

---

### Step 3: Install as systemd service

```
sudo ./svc.sh install
```

Expected output:

```
Creating service file...
Service installed successfully
```

---

### Step 4: Start the service

```
sudo ./svc.sh start
```

---

## 5️⃣ Validate Agent Is Running

```
systemctl status vsts.agent.*
```

Expected:

```
Active: active (running)
```

Also verify in **Azure DevOps**:

```
Project Settings → Agent Pools → Self_Agent_Pool → SonarQube = Online
```

---

## 6️⃣ If It Still Fails (Advanced Checks)

### Check service file exists

```
ls /etc/systemd/system/vsts.agent.*
```

### Reload systemd

```
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
```

### Check logs

```
journalctl -u vsts.agent.tejastarghale47.Self_Agent_Pool.SonarQube -xe
```

---

## 7️⃣ Common Mistakes (Seen Frequently)

```
❌ Agent extracted but never installed as service
❌ Agent renamed manually after install
❌ Multiple agents in same directory
❌ Running svc.sh from wrong folder
❌ Deleting /etc/systemd/system/vsts.agent*.service
```

---

## ✅ Final Recommendation (Best Practice)

```
One agent = one directory = one service
```

Example:

```
/home/test/agents/sonarqube-agent/
```

============================================== Adding Java Capability to the DevOps Agent ========================================================

Below is a **clear, root-cause analysis and resolution**, written in **Notepad++-style structured format**, aligned with Azure DevOps self-hosted agent internals.

---

## ERROR SUMMARY

##[error] No agent found in pool Self_Agent_Pool which satisfies the specified demands:

* java
* Agent.Version > 3.218.0

Agent Pool      : Self_Agent_Pool
Agent Name      : SonarQube
Agent Status    : Online / Idle
Agent Version   : 4.266.2 (✔ satisfies version demand)

---

## ROOT CAUSE (IMPORTANT)

Your **self-hosted agent does NOT advertise Java as a capability**.

Azure DevOps **does not check whether Java is needed** — it checks whether the agent **declares** it as a capability.

Even though:

* Agent is online ✔
* Agent version is correct ✔

The job fails because:
❌ **Java is not installed or not detectable on the agent host**
❌ OR Java is installed but **JAVA_HOME is not configured**
❌ OR the agent was started **before Java was installed**

---

## WHY THIS HAPPENS WITH SONARQUBE

SonarQube tasks automatically add this demand:

```
demands:
  - java
```

Azure DevOps then tries to find an agent that reports:

* java = true
* java path available

Your agent currently reports:
java : NOT FOUND

---

## VERIFY CURRENT AGENT CAPABILITIES

On Azure DevOps UI:

Agent Pool
→ Self_Agent_Pool
→ Agents
→ SonarQube
→ Capabilities tab

You will NOT see:
java
JAVA_HOME

---

## FIX OPTION 1 (RECOMMENDED) – INSTALL JAVA PROPERLY

On the **agent VM**:

### Step 1: Install Java (OpenJDK 11 or 17 recommended)

Ubuntu:

```
sudo apt update
sudo apt install -y openjdk-17-jdk
```

RHEL / Rocky / Alma:

```
sudo dnf install -y java-17-openjdk
```

### Step 2: Set JAVA_HOME

Find Java path:

```
readlink -f $(which java)
```

Example output:

```
/usr/lib/jvm/java-17-openjdk/bin/java
```

Set JAVA_HOME:

```
sudo vi /etc/profile.d/java.sh
```

Add:

```
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk
export PATH=$JAVA_HOME/bin:$PATH
```

Apply:

```
source /etc/profile.d/java.sh
```

Verify:

```
java -version
echo $JAVA_HOME
```

---

## CRITICAL STEP – RESTART AGENT (MANDATORY)

The agent detects capabilities **only at startup**.

From agent directory:

```
cd ~/agent
sudo ./svc.sh stop
sudo ./svc.sh start
```

OR (if not running as service):

```
./run.sh
```

---

## VERIFY FIX

Azure DevOps UI → Agent → Capabilities
You MUST now see:

```
java
JAVA_HOME
```

Then rerun pipeline.

---

## FIX OPTION 2 (TEMPORARY / NOT RECOMMENDED)

Remove Java demand manually (not advised for SonarQube):

```
pool:
  name: Self_Agent_Pool
  demands: []
```

⚠ SonarQube **will fail at runtime** without Java.

---

## WHY YOUR AGENT LOOKED “HEALTHY” BUT FAILED

Azure DevOps agent health ≠ job compatibility.

| Check           | Status |
| --------------- | ------ |
| Agent online    | ✔      |
| Agent enabled   | ✔      |
| Agent version   | ✔      |
| Java capability | ❌      |

---

## FINAL CONFIRMATION CHECKLIST

✔ Java installed
✔ JAVA_HOME set
✔ Agent restarted
✔ Java visible in agent capabilities
✔ Pipeline runs successfully

---

If you want, I can:

* Validate your **exact Linux distro**
* Provide **SonarQube-optimized Java version**
* Review your **YAML pipeline demands**
* Audit full **agent capability output**

State the next action you want.


=========================================== Adding Tools To Azure DevOps Agent ==============================

### Azure DevOps Pipeline Error — Python 3.12 Not Found on Self-Hosted Agent

*(Notepad++ format, root cause + remediation)*

---

#### 1. Problem Summary

Your pipeline step **“Use Python 3.12”** failed with the error:

```
Version spec 3.12 for architecture x64 did not match any version in Agent.ToolsDirectory
```

Agent tools directory:

```
/home/test/agent/_work/_tool
```

This occurred on a **self-hosted Linux agent** registered with **Azure DevOps**.

---

#### 2. Root Cause (Why This Happens)

On **self-hosted agents**, Azure DevOps **does NOT automatically install Python versions**.

The task:

```
UsePythonVersion@0
```

expects Python **3.12.x** to already exist in:

```
/home/test/agent/_work/_tool/Python/3.12.x/x64/
```

Your log confirms:

* Explicit version requested: `3.12.12`
* Agent.ToolsDirectory is empty (or lacks Python 3.12)
* Therefore, the task fails immediately

> Microsoft-hosted agents work differently — they come preloaded with multiple Python versions. Self-hosted agents do not.

---

#### 3. Quick Verification (Recommended)

Run this **on the agent VM**:

```bash
ls -R /home/test/agent/_work/_tool
```

Expected (but currently missing):

```
Python/
 └── 3.12.12/
     └── x64/
         ├── bin/python3
         └── ...
```

---

#### 4. Solution Options (Choose One)

---

### Option A — Install Python 3.12 Side-by-Side (Best Practice)

This is the **officially recommended approach**.

#### Step 1: Install build dependencies

```bash
sudo apt update
sudo apt install -y build-essential libssl-dev zlib1g-dev \
libncurses5-dev libncursesw5-dev libreadline-dev \
libsqlite3-dev libgdbm-dev libdb5.3-dev libbz2-dev \
libexpat1-dev liblzma-dev tk-dev wget
```

#### Step 2: Download & build Python 3.12

```bash
cd /tmp
wget https://www.python.org/ftp/python/3.12.12/Python-3.12.12.tgz
tar -xvf Python-3.12.12.tgz
cd Python-3.12.12

./configure --enable-optimizations
make -j$(nproc)
sudo make altinstall
```

Verify:

```bash
python3.12 --version
```

---

#### Step 3: Register Python with Azure DevOps Agent

```bash
mkdir -p /home/test/agent/_work/_tool/Python/3.12.12/x64
ln -s /usr/local/bin/python3.12 \
/home/test/agent/_work/_tool/Python/3.12.12/x64/python
```

Restart agent:

```bash
cd /home/test/agent
sudo ./svc.sh restart
```

---

#### Step 4: Pipeline YAML (Correct)

```yaml
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.12'
    architecture: 'x64'
```

---

### Option B — Use System Python (Fastest, Less Clean)

If Python 3.12 is already installed system-wide:

```yaml
- script: |
    python3 --version
  displayName: "Verify system Python"
```

⚠ Limitation:

* Bypasses `UsePythonVersion`
* Not portable across agents
* Can break future pipelines

---

### Option C — Downgrade Python Version (If Allowed)

If your project does **not strictly require 3.12**:

```yaml
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.10'
```

(Only valid if Python 3.10 already exists under `_tool`)

---

#### 5. Why Your Agent Failed but Microsoft Agent Would Not

| Feature                | Microsoft-Hosted | Self-Hosted |
| ---------------------- | ---------------- | ----------- |
| Python preinstalled    | Yes              | No          |
| `_tool` auto-populated | Yes              | No          |
| Side-by-side versions  | Managed          | Manual      |
| Maintenance effort     | None             | Yours       |

---

#### 6. Final Recommendation

For a **production-grade DevOps setup** (especially for Django + CI pipelines):

✔ Install Python 3.12 side-by-side
✔ Register it under `Agent.ToolsDirectory`
✔ Keep `UsePythonVersion@0` in YAML

This ensures:

* Deterministic builds
* Version pinning
* Compatibility with future pipelines

---

===================== Difference Between NAT, Bridge, and Host-Only ==================

Below is a **clear, conceptual explanation** of **NAT**, **Bridged**, and **Host-Only** networking in **VirtualBox**, written to help you reason about **when and why** to use each mode rather than just memorizing options.

![Image](https://techbeatly.com/images/tb-uploads/2019/07/VirtualBox-network-modes-how-the-NAT-mode-works.png?utm_source=chatgpt.com)

![Image](https://i.sstatic.net/I5qvT.png?utm_source=chatgpt.com)

![Image](https://gotocloud.co.kr/wp-content/uploads/2024/01/Host-only_Network.jpg?utm_source=chatgpt.com)

---

## 1. NAT (Network Address Translation)

### Core Concept

**NAT hides the VM behind the host machine**, similar to how your home router hides devices behind one public IP.

The VM:

* Uses a **private IP** (e.g., `10.0.2.x`)
* Sends traffic **out via the host**
* **Cannot receive inbound traffic by default**

```
VM ──► Host ──► Internet
```

### Key Characteristics

* VirtualBox runs an internal router
* DHCP, DNS, and gateway are auto-provided
* Outbound traffic only
* Inbound access requires **port forwarding**

### Practical Meaning

* VM can download packages, pull Docker images, access APIs
* Other machines **cannot see or reach the VM**

### Typical Use Cases

* Learning Linux
* CI/CD agents
* Terraform, Ansible, Docker labs
* Secure, internet-only access

### Mental Model

> “My VM behaves like a laptop behind a home router.”

---

## 2. Bridged Adapter

### Core Concept

**The VM becomes a full peer on your physical network**, as if it were a real machine plugged into the same switch or Wi-Fi.

```
VM ──► Router ──► Internet
Host ─► Router
```

### Key Characteristics

* VM gets IP from your LAN DHCP (e.g., `192.168.x.x`)
* Same subnet as host
* Fully reachable from LAN
* Subject to LAN security policies

### Practical Meaning

* SSH, HTTP, databases work without port forwarding
* VM visible to routers, firewalls, and other devices

### Typical Use Cases

* Server simulations
* Web app testing from mobile devices
* Monitoring tools (Prometheus, Zabbix, SCOM)
* On-prem-like environments

### Mental Model

> “My VM is another physical server on my network.”

---

## 3. Host-Only Network

### Core Concept

**A private network exists only between the host and the VM.**
No internet. No LAN access.

```
Host ◄──► VM
(no Internet)
```

### Key Characteristics

* Dedicated virtual subnet (e.g., `192.168.56.x`)
* Host ↔ VM communication only
* Fully isolated from external networks

### Practical Meaning

* SSH always works from host
* Safe testing environment
* Predictable IPs

### Typical Use Cases

* Security labs
* Database testing
* Isolated automation experiments
* Teaching networking concepts

### Mental Model

> “A private Ethernet cable between my laptop and VM.”

---

## Comparison Table (At a Glance)

| Feature                    | NAT              | Bridged | Host-Only |
| -------------------------- | ---------------- | ------- | --------- |
| Internet access            | ✔                | ✔       | ✖         |
| Host → VM SSH              | ✖ (port forward) | ✔       | ✔         |
| LAN → VM access            | ✖                | ✔       | ✖         |
| VM visible on LAN          | ✖                | ✔       | ✖         |
| Security                   | High             | Medium  | Very High |
| Corporate network friendly | ✔                | ✖       | ✔         |

---

## Real-World Analogy

| VirtualBox Mode | Real-World Equivalent           |
| --------------- | ------------------------------- |
| NAT             | Laptop behind home router       |
| Bridged         | Physical server on office LAN   |
| Host-Only       | Direct Ethernet cable to laptop |

---

## Best-Practice Recommendation (DevOps Labs)

**Use TWO adapters:**

* **Adapter 1:** NAT (internet, updates, downloads)
* **Adapter 2:** Host-Only (SSH, automation, testing)

This gives:

* Internet access
* Stable SSH
* No LAN exposure
* Works even with VPNs

---

## Summary (Decision Rule)

* Need **internet only** → **NAT**
* Need **LAN visibility** → **Bridged**
* Need **isolation** → **Host-Only**
* Need **everything safely** → **NAT + Host-Only**

=======================================================================================