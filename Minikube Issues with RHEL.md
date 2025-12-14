
[test@localhost ~]$ minikube start
* minikube v1.37.0 on Redhat 9.7
* Unable to pick a default driver. Here is what was considered, in preference order:
  - podman: Not healthy: "sudo -n -k podman version --format {{.Version}}" exit status 1: sudo: a password is required
  - podman: Suggestion: Add your user to the 'sudoers' file: 'test ALL=(ALL) NOPASSWD: /usr/bin/podman' , or run 'minikube config set rootless true' <https://podman.io>
* Alternatively you could install one of these drivers:
  - docker: Not installed: exec: "docker": executable file not found in $PATH
  - kvm2: Not installed: exec: "virsh": executable file not found in $PATH
  - qemu2: Not installed: exec: "qemu-system-x86_64": executable file not found in $PATH
  - virtualbox: Not installed: unable to find VBoxManage in $PATH

X Exiting due to DRV_NOT_HEALTHY: Found driver(s) but none were healthy. See above for suggestions how to fix installed drivers.

test ALL=(ALL) NOPASSWD: /usr/bin/podman

minikube config set rootless true


===================================== Issues fixing with Minikube starting at RHEL 9 =====================================

You’re actually very close now – this time it’s failing due to **rootless cgroup delegation** and a leftover Podman volume.

I’ll break it into clear steps you can just run.

---

==========================================

1. What is failing now (in simple terms)
   ==========================================

From your output:

* Driver:
  `MINIKUBE_ROOTLESS=true` → **rootless Podman driver** selected.
* Main failure:

  ```text
  ERROR: UserNS: cpu controller needs to be delegated
  ...
  Failed to start podman container...
  Error: volume with name minikube already exists: volume already exists
  ```

This means:

1. Rootless containers don’t have the **cpu cgroup controller delegated** to your user.
2. minikube retried and then hit a second error because Podman **volume `minikube` already exists** from the previous failed attempt.

This is a known issue with rootless Podman + minikube; the fix is to **delegate cgroup controllers via systemd** and then clean up the old minikube/Podman state. ([GitHub][1])

---

=====================================================
2. Fix cgroup delegation for rootless Podman (one-time)
=======================================================

Run these commands as `root` (or with `sudo`) on your Red Hat 9.7 host:

```bash
sudo mkdir -p /etc/systemd/system/user@.service.d

cat <<EOF | sudo tee /etc/systemd/system/user@.service.d/delegate.conf
[Service]
Delegate=cpu cpuset io memory pids
EOF

sudo systemctl daemon-reload
```

Explanation:

* This creates a **systemd drop-in** for `user@.service` which tells systemd to delegate the `cpu`, `cpuset`, `io`, `memory`, and `pids` controllers to user sessions. ([GitHub][1])
* Rootless Podman (and thus minikube) then gets proper cgroup access.

Now you **must log out and log back in** as user `test` so your user session picks up the new systemd configuration.

---

===================================
3. Clean up previous failed state
=================================

After logging back in as `test`, run:

```bash
# Remove any existing minikube cluster
minikube delete --all --purge

# Remove leftover Podman volume causing "volume already exists"
podman volume rm minikube || true
```

* `minikube delete --all --purge` ensures old profiles/configs are wiped.
* `podman volume rm minikube` removes the conflicting volume that minikube is complaining about.

You can verify:

```bash
podman volume ls | grep minikube
# should show nothing
```

---

================================================
4. Start minikube again with safer memory limits
================================================

You have **~3.6 GiB** total RAM. Requesting 3072MB is very tight, so use something like **2048MB–2200MB**.

From user `test`:

```bash
# Optional but recommended: ensure driver and rootless settings are explicit
minikube config set driver podman
minikube config set rootless true

# Now start with lower memory and containerd runtime
minikube start \
  --driver=podman \
  --container-runtime=containerd \
  --cpus=2 \
  --memory=2200mb
```

Notes:

* `--container-runtime=containerd` is the runtime recommended in the upstream workaround for rootless Podman + minikube. ([GitHub][1])
* `--memory=2200mb` should reduce pressure on the host and avoid stability issues.

You may still see a **warning** about memory, but this should not stop minikube from starting as long as the podman container can be created.

---

===================================================
5. If you want to stop using rootless mode altogether
=====================================================

If rootless setup still feels too fragile, you can instead run minikube with **rootful Podman**:

1. Make sure you’re not forcing rootless:

   ```bash
   unset MINIKUBE_ROOTLESS     # if you had it exported
   minikube config unset rootless 2>/dev/null || true
   ```

2. As root (or with sudo) run:

   ```bash
   sudo minikube delete --all --purge
   sudo podman volume rm minikube || true

   sudo minikube start \
     --driver=podman \
     --container-runtime=containerd \
     --cpus=2 \
     --memory=2200mb
   ```

Running minikube under sudo is simpler because systemd delegation and user-namespace constraints are less strict, but it’s less “rootless”.

---

=========================================
6. Quick checklist to confirm it worked
=======================================

After a successful start, these should work as `test`:

```bash
minikube status
kubectl get nodes
```

* `minikube status` should show the host, kubelet, and apiserver as “Running”.
* `kubectl get nodes` should list a single `minikube` node with `STATUS=Ready`.

---

If you run into any new error lines after applying the delegation fix and cleanup, paste just the last `minikube start` output and we can fine-tune from there.

[1]: https://github.com/kubernetes/minikube/issues/15697?utm_source=chatgpt.com "oci.PrepareContainerNode() fails in subsequent calls with ..."

==========================================================================================================================