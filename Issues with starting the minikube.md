Good — this is the same `juju` lock permission problem. Let’s apply the proven fixes, verify the environment, clean any stale lock files, and re-run `minikube start`. Copy-paste the commands below in order.

> **Important security note:** `fs.protected_regular=0` relaxes a kernel protection on `/tmp`. If this host is multi-user or production, prefer the Docker driver or re-enable the protection after finishing. I’ll show how to re-enable it below.


# 1) Quick check — current value

```bash
# show current kernel setting
sysctl fs.protected_regular
# expected output: fs.protected_regular = 1  (or 0)
```

---

# 2) Temporary fix (takes effect immediately)

```bash
# as root
sudo sysctl -w fs.protected_regular=0

# verify
sysctl fs.protected_regular
# should show: fs.protected_regular = 0
```

---

# 3) Remove stale juju/minikube tmp lock files (safe)

```bash
# list juju/minikube tmp files first
ls -la /tmp/juju-* /tmp/mk* 2>/dev/null || true

# remove them (run as root)
sudo rm -f /tmp/juju-* /tmp/mk*  /tmp/mkb* /tmp/mk* 2>/dev/null || true
```

---

# 4) Confirm /tmp permissions and mount options

```bash
# ensure /tmp is world-writable with sticky bit
ls -ld /tmp

# check mount options
mount | grep " on /tmp "
```

`/tmp` should look like: `drwxrwxrwt` (note the trailing `t`). If not, run:

```bash
sudo chmod 1777 /tmp
```

---

# 5) Re-run minikube start (same command you used)

```bash
sudo minikube start \
  --driver=podman \
  --container-runtime=containerd \
  --cpus=2 \
  --memory=2200mb \
  --force
```

If it proceeds past the previous error, great — continue with enabling addons.

---

# 6) Make the kernel change persistent (optional)

If you need the change to survive reboots:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/99-minikube-juju.conf
# Allow minikube/juju temp lockfiles to be opened across users
fs.protected_regular = 0
EOF

sudo sysctl --system
```

---

# 7) How to revert (recommended after you finish)

```bash
# re-enable protection immediately
sudo sysctl -w fs.protected_regular=1

# remove persistent file if created
sudo rm -f /etc/sysctl.d/99-minikube-juju.conf
sudo sysctl --system
```

---

# 8) If the problem persists — additional diagnostics

If `minikube start` still fails after steps 2–5, capture these and paste them here:

```bash
# kernel setting
sysctl fs.protected_regular

# permissions and files in /tmp
ls -la /tmp | head -n 50
ls -la /tmp/juju-* /tmp/mk* 2>/dev/null || true

# journal entries around the failure time
journalctl -u crio -n 200 --no-pager || journalctl -n 200 --no-pager

# minikube debug logs
minikube logs --file=minikube-logs.txt
tail -n 200 minikube-logs.txt
```

Also, if SELinux is enforcing and you still get `permission denied`, check for AVC denies:

```bash
sudo ausearch -m avc -ts recent
sudo sealert -a /var/log/audit/audit.log    # if setools installed
```

(We can interpret those outputs and adjust.)

---

# 9) Safer alternative (if you prefer *not* to change kernel settings)

* Install Docker and run `minikube start --driver=docker` as your normal user. That avoids `fs.protected_regular` issues entirely because minikube runs as non-root against docker.

---

Run steps 2–5 now and paste the tail of the `minikube start` output if it still fails. I’ll analyze the exact failure lines and give the next corrective commands.
