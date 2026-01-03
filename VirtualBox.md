https://www.activecountermeasures.com/port-forwarding-with-virtualbox/?utm_source=chatgpt.com

Understood. **NAT should provide internet access automatically**, so if it does not, there is a **misconfiguration either in VirtualBox or inside Ubuntu**. Below is a **systematic, production-style troubleshooting checklist**. Follow it in order.

---

## 1. Verify VirtualBox NAT Configuration (Host Side)

### Step 1.1 – Adapter settings

In **VirtualBox Manager**:

* VM → **Settings** → **Network**
* **Adapter 1**

  * ✔ Enable Network Adapter
  * ✔ Attached to: **NAT**
  * ✔ Name: (leave default)
  * ✔ Cable Connected: **Checked**

➡ If “Cable Connected” is unchecked, the VM will never get internet.

---

## 2. Check Network Interface Inside Ubuntu

Login to Ubuntu VM and run:

```bash
ip addr
```

You should see something like:

```
enp0s3
inet 10.0.2.15/24
```

### Expected for NAT

* Interface name: `enp0s3` (most common)
* IP range: `10.0.2.x`

### If you see:

* ❌ **No IP address** → DHCP failure
* ❌ Interface DOWN → Network not started

Bring interface up manually:

```bash
sudo ip link set enp0s3 up
sudo dhclient enp0s3
```

Re-check:

```bash
ip addr
```

---

## 3. Verify Default Route (Critical)

Run:

```bash
ip route
```

Expected output:

```
default via 10.0.2.2 dev enp0s3
```

### If default route is missing:

Add it manually:

```bash
sudo ip route add default via 10.0.2.2 dev enp0s3
```

Then test connectivity.

---

## 4. Test Connectivity Step-by-Step (Isolate the Failure)

### Step 4.1 – Test VirtualBox NAT Gateway

```bash
ping -c 3 10.0.2.2
```

✔ Success → NAT gateway reachable
❌ Failure → VirtualBox NAT issue

---

### Step 4.2 – Test Public IP (Bypass DNS)

```bash
ping -c 3 8.8.8.8
```

* ✔ Works → Internet OK, DNS issue
* ❌ Fails → Internet blocked

---

### Step 4.3 – Test DNS

```bash
ping -c 3 google.com
```

If IP ping works but DNS fails → DNS misconfiguration.

---

## 5. Fix DNS Issues (Most Common Problem)

Check DNS:

```bash
cat /etc/resolv.conf
```

Expected:

```
nameserver 127.0.0.53
```

or

```
nameserver 10.0.2.3
```

### Quick Fix (Temporary)

```bash
sudo nano /etc/resolv.conf
```

Replace contents with:

```
nameserver 8.8.8.8
nameserver 1.1.1.1
```

Test again:

```bash
ping google.com
```

---

## 6. Restart Network Services (Ubuntu)

```bash
sudo systemctl restart NetworkManager
```

or (older systems):

```bash
sudo systemctl restart networking
```

---

## 7. Check Firewall (Rare but Possible)

```bash
sudo ufw status
```

If active, temporarily disable:

```bash
sudo ufw disable
```

Test internet again.

---

## 8. Corporate / VPN / Proxy Interference (Very Common)

If **any of these are true**:

* Host machine is connected to **corporate VPN**
* Host uses **proxy**
* Office Wi-Fi with restrictions

Then NAT may silently fail.

### Quick test

* Disconnect VPN on host
* Restart VM
* Test internet again

---

## 9. Quick Recovery Method (Works 90% of Time)

If still broken:

1. Power off VM
2. VirtualBox → **File → Tools → Network Manager**
3. **NAT Networks**

   * Delete existing NAT network
   * Create new NAT network (DHCP ON)
4. Attach VM to this new NAT network
5. Start VM

---

## 10. Validation Command (Final Check)

```bash
curl -I https://google.com
```

Expected:

```
HTTP/2 200
```

---

## Summary (Root Cause Probability)

| Cause                      | Probability |
| -------------------------- | ----------- |
| DNS misconfiguration       | Very High   |
| VPN / Proxy on host        | High        |
| Missing default route      | Medium      |
| VirtualBox adapter mis-set | Medium      |
| Firewall                   | Low         |

---

### Recommended for DevOps Labs (Stable Setup)

If NAT keeps failing in your environment:

* **Adapter 1:** NAT (Internet)
* **Adapter 2:** Host-Only (SSH access)

This avoids corporate/VPN issues while keeping internet.

==================================== SSH Generally Would not happens over NAT ============================

This is **expected behavior** with **NAT**. Nothing is broken.

With **VirtualBox NAT**, the VM **cannot be reached directly** from the host or LAN **unless you configure port forwarding**.

Below is the **correct, production-grade way** to take an SSH session when using NAT.

![Image](https://bobcares.com/wp-content/uploads/Virtualbox_ssh_nat.png?utm_source=chatgpt.com)

![Image](https://www.activecountermeasures.com/wp-content/uploads/2023/02/blog-feature-3-Port-Forwarding-With-VirtualBox.png?utm_source=chatgpt.com)

![Image](https://techbeatly.com/images/tb-uploads/2019/07/VirtualBox-network-modes-how-the-NAT-mode-works.png?utm_source=chatgpt.com)

---

## Why SSH Does Not Work with NAT (By Design)

* NAT hides the VM behind the host
* VM gets a private IP (e.g., `10.0.2.15`)
* Inbound connections (SSH) are **blocked**
* Only **outbound** traffic works by default

✔ Internet works
❌ SSH fails (unless port forwarding is configured)

---

## Option 1 (Recommended): NAT + Port Forwarding (Best Practice)

### Step 1 – Configure Port Forwarding

**VirtualBox Manager → VM → Settings → Network → Adapter 1 (NAT)**
Click **Advanced → Port Forwarding**

Add rule:

| Field      | Value         |
| ---------- | ------------- |
| Name       | SSH           |
| Protocol   | TCP           |
| Host IP    | 127.0.0.1     |
| Host Port  | 2222          |
| Guest IP   | (leave empty) |
| Guest Port | 22            |

Click **OK → OK**

---

### Step 2 – Ensure SSH Server Is Installed in Ubuntu

Inside VM:

```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

Verify:

```bash
systemctl status ssh
```

---

### Step 3 – SSH from Host Machine

From **host**:

```bash
ssh -p 2222 username@127.0.0.1
```

Example:

```bash
ssh -p 2222 tejas@127.0.0.1
```

✔ This is the **correct way** to SSH with NAT.

---

## Option 2: Switch to Bridged Adapter (Simpler, Less Secure)

If you want **direct SSH without port forwarding**:

**VirtualBox → Settings → Network**

* Attached to: **Bridged Adapter**
* Select active NIC (Wi-Fi/Ethernet)

VM gets LAN IP (e.g., `192.168.x.x`)

SSH:

```bash
ssh username@192.168.x.x
```

⚠ May fail on corporate networks.

---

## Option 3 (Most Stable for Labs): NAT + Host-Only (Two Adapters)

### Configuration

* **Adapter 1:** NAT (Internet)
* **Adapter 2:** Host-Only (SSH)

Inside VM:

```bash
ip addr
```

Example:

```
enp0s3 → 10.0.2.15 (NAT)
enp0s8 → 192.168.56.101 (Host-Only)
```

SSH:

```bash
ssh username@192.168.56.101
```

✔ No port forwarding
✔ No LAN exposure
✔ Works even on VPN/corporate Wi-Fi

**This is the preferred DevOps lab setup.**

---

## Common Mistakes (Very Important)

❌ Trying to SSH to `10.0.2.15` from host
❌ Forgetting port forwarding
❌ SSH service not installed
❌ Firewall blocking port 22

Check firewall:

```bash
sudo ufw allow ssh
sudo ufw reload
```

---

## Decision Summary

| Requirement    | Best Option           |
| -------------- | --------------------- |
| Internet only  | NAT                   |
| SSH with NAT   | NAT + Port Forwarding |
| Direct LAN SSH | Bridged               |
| DevOps labs    | NAT + Host-Only       |

---

### My Recommendation for You

Given your **DevOps / CI/CD / automation** usage:

* Use **NAT + Host-Only**
* Avoid Bridged on corporate networks
* Avoid port forwarding complexity long-term

---

If you want, I can provide:

* Exact **two-adapter configuration screenshots**
* **Static IP config** for Host-Only
* SSH hardening (keys, non-root, firewall)
* Multi-VM SSH access setup (Ansible-ready)

========================= Fixing the DHCP Assignment Issues ===========================

============================================================
**Root Cause Identified – Devices Are “Strictly Unmanaged”**
============================================================

Option A (Recommended): Use NetworkManager (Already Installed)

Ubuntu uses NetworkManager by default. Do this:

1) Check device status
nmcli device status


You should see:

enp0s8   ethernet   disconnected

2) Bring the interface up using DHCP
sudo nmcli device connect enp0s8

3) Verify IPv4 address
ip addr show enp0s8

============================================================

Your output is conclusive:

```
enp0s3  ethernet  unmanaged
enp0s8  ethernet  unmanaged
```

This means **NetworkManager is NOT managing any interfaces on this VM**.
Therefore:

• `nmcli` **cannot** bring the interface up
• DHCP via NetworkManager **will never work**
• This is **by design**, not an error

Ubuntu is using **Netplan + systemd-networkd**, not NetworkManager.

---

## **Why This Happens (Important Concept)**

On server/minimal Ubuntu installs:

• NetworkManager is disabled
• Netplan delegates networking to `systemd-networkd`
• All interfaces appear as **unmanaged** in `nmcli`

So the correct fix is **Netplan**, not `nmcli`.

---

## **Correct Fix – Configure enp0s8 with Netplan (DHCP)**

### 1) Identify Netplan file

```
ls /etc/netplan/
```

Typical file:

```
00-installer-config.yaml
or
50-cloud-init.yaml
```

---

### 2) Edit the Netplan configuration

```
sudo nano /etc/netplan/*.yaml
```

Example **correct configuration** for your setup:

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: true
```

IMPORTANT:
• YAML is indentation-sensitive
• Use spaces, not tabs

---

### 3) Apply the configuration

```
sudo netplan generate
sudo netplan apply
```

---

## **4) Verify IP Assignment**

```
ip addr show enp0s8
```

Expected:

```
inet 192.168.56.x/24
```

---

## **5) Verify Routing**

```
ip route
```

You should see:

```
default via 10.0.2.2 dev enp0s3
192.168.56.0/24 dev enp0s8
```

---

## **6) SSH Will Now Work**

From the host:

```
ssh vboxuser@192.168.56.x
```

---

## **Why This Is the Correct Architecture**

| Interface | Purpose         | Managed By       |
| --------- | --------------- | ---------------- |
| enp0s3    | Internet (NAT)  | systemd-networkd |
| enp0s8    | Host-only (SSH) | systemd-networkd |

This is **exactly how Ubuntu Server networking is intended to work**.

---

## **Final Verdict**

✔ Interface is UP
✔ VirtualBox is configured correctly
✔ Issue was NetworkManager not in use
✔ Netplan is the authoritative fix
