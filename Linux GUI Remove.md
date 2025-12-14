📌 **Disable / Enable GUI in Red Hat Enterprise Linux (RHEL)**
(Works for RHEL 7 / 8 / 9 – systemd-based)

🔥 There are **2 recommended methods**:

===========================================================

### **METHOD 1️⃣ — Change System Boot Target (Recommended)**

# (Keep GUI installed but start only CLI on boot)

Check Current Boot Target

```
systemctl get-default
```

• `graphical.target` = GUI Mode
• `multi-user.target` = CLI Mode

---

Disable GUI (Boot into CLI mode permanently)

```
sudo systemctl set-default multi-user.target
sudo systemctl isolate multi-user.target
```

• `isolate` switches to CLI immediately
• Target remains CLI after reboot

---

▶ **Enable GUI temporarily (only for this session)**

```
sudo systemctl isolate graphical.target
```

(Reverts to CLI after reboot if default is still multi-user.target)

---

**Enable GUI permanently (Start GUI on boot)**

```
sudo systemctl set-default graphical.target
sudo systemctl isolate graphical.target
```

===========================================================

**METHOD 2️⃣ — Remove GUI Packages Completely**

# (For servers needing more performance + less attack surface)

⚠ This is permanent unless reinstalled.

---

**Check installed environment**

```
sudo yum group list --installed
```

---

**Install Minimal Base (Ensure CLI packages exist)**

```
sudo yum groupinstall "Server"
```

or

```
sudo yum groupinstall "Minimal Install"
```

---

**Remove GUI desktop group**

```
sudo yum groupremove "Server with GUI"
```

Optional (Remove GNOME Display Manager if still present):

```
sudo systemctl disable gdm
sudo yum remove gdm
```

---

**Reboot system**

```
sudo reboot
```

===========================================================

### **Useful Additional Commands**

===========================================================

**Switch to CLI for current boot only**

```
sudo systemctl isolate multi-user.target
```

**Switch to GUI for current boot only**

```
sudo systemctl isolate graphical.target
```

**List all systemd targets**

```
systemctl list-units --type=target
```
**Check graphical login service status**

```
systemctl status gdm
```

===========================================================
✔ **Summary**
=============

| Action              | Command                                   |
| ------------------- | ----------------------------------------- |
| Boot default to CLI | `systemctl set-default multi-user.target` |
| Boot default to GUI | `systemctl set-default graphical.target`  |
| Remove GUI fully    | `yum groupremove "Server with GUI"`       |

---

## 📌 End of Notepad++ Document