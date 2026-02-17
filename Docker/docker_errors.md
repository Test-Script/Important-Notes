Below is a **clear root-cause explanation and corrective procedure**, formatted for direct use in **Notepad++**.

---

## ISSUE SUMMARY

You are receiving the error:

permission denied while trying to connect to the Docker daemon socket
unix:///var/run/docker.sock

This occurs **even after** running:

sudo usermod -aG docker $USER

---

## ROOT CAUSE (IMPORTANT)

Adding a user to the `docker` group **does NOT take effect in the current login session**.

Your shell is still operating without the updated group membership, so Docker continues to deny access to `/var/run/docker.sock`.

---

## IMMEDIATE FIX (RECOMMENDED)

Option 1 (Fastest – no reboot):

```bash
newgrp docker
docker compose up -d
```

This reloads group membership for the current shell only.

---

## PERMANENT FIX (BEST PRACTICE)

1. Log out completely
2. Log back in
3. Verify group membership:

```bash
groups
```

You must see `docker` in the output.

Then run:

```bash
docker compose up -d
```

---

## VERIFY DOCKER DAEMON STATUS

Before retrying, confirm Docker is running:

```bash
sudo systemctl status docker
```

If not running:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

---

## VALIDATE SOCKET PERMISSIONS

```bash
ls -l /var/run/docker.sock
```

Expected output:

```text
srw-rw---- 1 root docker ...
```

If permissions are incorrect (rare):

```bash
sudo chown root:docker /var/run/docker.sock
sudo chmod 660 /var/run/docker.sock
```

---

## ABOUT THE COMPOSE WARNING (SAFE TO IGNORE)

```text
the attribute `version` is obsolete
```

This warning is **non-fatal**.
Docker Compose v2 ignores the `version:` field.

Optional cleanup:

```yaml
# Remove this line
version: "3.8"
```

---

## FINAL EXPECTED RESULT

After applying **newgrp** or re-login:

```bash
docker compose up -d
```

Should successfully pull:

* sonarqube:community
* postgres:12

And start containers without permission errors.

---

## SECURITY NOTE

Running Docker without `sudo` is expected **only** when:

* Docker daemon is running
* User is in `docker` group
* Session is refreshed

---

## NEXT STEP (OPTIONAL)

Once containers are up, confirm SonarQube:

```bash
docker ps
docker logs sonarqube
```

Default URL:
[http://localhost:9000](http://localhost:9000)

---
