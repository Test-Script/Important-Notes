Below is a **clear, enterprise-ready guide** for **running Trivy using Docker**, formatted in a **Notepad++–friendly structure** (plain text, clean sections, copy-paste ready).

==================================================
TRIVY – RUNNING WITH DOCKER (IMAGE, FS, CONFIG)
===============================================

![Image](https://devopscube.com/content/images/2025/03/trivy-blog-image-1.gif)

![Image](https://www.aquasec.com/wp-content/uploads/2024/01/Picture-4-Docker-Desktop-Dashboard-Trivy-Extension-%E2%80%93-Image-Scan-and-Vulnerability-list.jpg)

![Image](https://trivy.dev/docs/latest/imgs/trivy-k8s.png)

---

1. WHAT IS TRIVY (IN DOCKER CONTEXT)

---

Trivy is a **containerized vulnerability scanner** that can scan:

* Docker images
* Local filesystems
* IaC (Terraform, Kubernetes, Helm)
* OS packages and application dependencies

Running Trivy via Docker is preferred in:

* CI/CD pipelines
* Hardened environments
* Systems without local Trivy installation

---

2. PREREQUISITES

---

* Docker installed and running
* Internet access (for vulnerability DB download)
* Docker socket access (for image scans)

Verify:

```
docker --version
docker ps
```

---

3. PULL TRIVY DOCKER IMAGE

---

```
docker pull aquasec/trivy:latest
```

(Optional: pin version for enterprise stability)

```
docker pull aquasec/trivy:0.49.1
```

---

4. SCAN A DOCKER IMAGE (MOST COMMON)

---

Example: Scan NGINX image

```
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image nginx:latest
```

Explanation:

* `/var/run/docker.sock` → allows Trivy to inspect local images
* `--rm` → auto cleanup container

---

5. FILTER BY SEVERITY (ENTERPRISE STANDARD)

---

```
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image \
  --severity HIGH,CRITICAL \
  nginx:latest
```

---

6. FAIL BUILD ON VULNERABILITIES (CI/CD)

---

```
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image \
  --severity CRITICAL \
  --exit-code 1 \
  nginx:latest
```

Use case:

* CI pipeline fails if CRITICAL vulnerabilities exist

---

7. SCAN LOCAL FILESYSTEM (SOURCE CODE / VM)

---

```
docker run --rm \
  -v $(pwd):/scan \
  aquasec/trivy fs /scan
```

Common usage:

* Scan application source before Docker build
* Detect secrets, misconfigurations

---

8. SCAN TERRAFORM / K8s / HELM (IaC)

---

Terraform example:

```
docker run --rm \
  -v $(pwd):/scan \
  aquasec/trivy config /scan
```

Detects:

* Public S3 buckets
* Open security groups
* Privileged containers
* Weak RBAC

---

9. GENERATE REPORTS (JSON / SARIF)

---

JSON (for dashboards):

```
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(pwd):/output \
  aquasec/trivy image \
  --format json \
  --output /output/trivy-report.json \
  nginx:latest
```

SARIF (GitHub Security tab):

```
--format sarif --output trivy.sarif
```

---

10. CACHE DB FOR FASTER SCANS (BEST PRACTICE)

---

```
docker volume create trivy-cache
```

```
docker run --rm \
  -v trivy-cache:/root/.cache/ \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image nginx:latest
```

Benefits:

* Faster scans
* Reduced network calls
* CI/CD friendly

---

11. AIR-GAPPED / OFFLINE ENVIRONMENTS

---

```
--skip-db-update
--offline-scan
```

Used in:

* Regulated environments
* Restricted networks

---

12. RECOMMENDED ENTERPRISE BASELINE COMMAND

---

```
docker run --rm \
  -v trivy-cache:/root/.cache/ \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image \
  --severity HIGH,CRITICAL \
  --exit-code 1 \
  --ignore-unfixed \
  myapp:latest
```

---

13. COMMON ISSUES & FIXES

---

Issue:
`permission denied /var/run/docker.sock`

Fix:

```
sudo usermod -aG docker $USER
newgrp docker
```

Issue: Slow scans
Fix:

* Use cache volume
* Pin Trivy version

---

14. NEXT (OPTIONAL – I CAN HELP)

---

* Trivy + GitHub Actions
* Trivy + GitLab CI
* Trivy + Jenkins
* Trivy + SonarQube (DevSecOps)
* Policy-based scanning (OPA / Rego)

======================================= Scanning Time Reduced ============================================

Below is a **concise, enterprise-grade guide** to **minimize Trivy scan time**, written in **Notepad++–friendly format** and focused on **real, measurable optimizations** used in CI/CD and production pipelines.

==================================================
HOW TO MINIMIZE TRIVY SCAN TIME (PRACTICAL)
===========================================

![Image](https://devopscube.com/content/images/2025/03/trivy-blog-image-1.gif)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/0%2A5JD3iiiggkEER4w8)

![Image](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2020/06/26/CICD-Container-Scannning-Figure-1s.png)

![Image](https://trivy.dev/docs/latest/imgs/client-server.png)

---

1. ENABLE VULNERABILITY DB CACHING (MOST IMPORTANT)

---

Without cache, Trivy downloads the DB every run.

Create cache volume (once):

```
docker volume create trivy-cache
```

Use cache:

```
docker run --rm \
  -v trivy-cache:/root/.cache \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image nginx:latest
```

Impact:

* Reduces scan time by **60–80%**
* Essential for CI/CD

---

2. SKIP DB UPDATE WHEN POSSIBLE

---

Use when:

* CI jobs run frequently
* DB already updated recently

```
--skip-db-update
```

Example:

```
aquasec/trivy image --skip-db-update myapp:latest
```

Impact:

* Saves 10–30 seconds per scan

---

3. SCAN ONLY REQUIRED SEVERITIES

---

Avoid scanning LOW / MEDIUM if not required.

```
--severity HIGH,CRITICAL
```

Why:

* Fewer checks
* Faster result processing
* Aligns with enterprise security baselines

---

4. IGNORE UNFIXED VULNERABILITIES

---

Unfixed CVEs increase scan time and noise.

```
--ignore-unfixed
```

Impact:

* Faster execution
* Cleaner reports
* CI-friendly

---

5. DISABLE UNUSED SCANNERS

---

By default, Trivy scans:

* OS packages
* Application dependencies
* Secrets
* Misconfigurations

Disable what you don’t need:

```
--scanners vuln
```

Examples:

* Image scan only:

```
--scanners vuln
```

* FS scan without secrets:

```
--scanners vuln,config
```

Impact:

* **Huge improvement** for source scans

---

6. USE `--timeout` TO AVOID LONG HANGS

---

```
--timeout 5m
```

Prevents:

* Hung pipelines
* Stuck registry pulls

---

7. SCAN SMALLER IMAGES (STRATEGIC)

---

Image choice directly affects scan time.

Best practices:

* Use `distroless`, `alpine`, or `ubi-micro`
* Multi-stage builds
* Remove package managers in final image

Example:

```
python:3.12-slim  →  40–50% faster than full image
```

---

8. USE `--light` MODE (FASTEST OPTION)

---

```
--light
```

What it does:

* Skips deep dependency analysis
* Uses lightweight vulnerability detection

Use case:

* Pre-merge checks
* Fast feedback pipelines

---

9. PARALLELIZE SCANS (CI OPTIMIZATION)

---

Instead of:

* One job scanning multiple images

Do:

* One image per job (parallel)

Example:

* frontend image job
* backend image job
* worker image job

Impact:

* Total pipeline time drastically reduced

---

10. AVOID FILESYSTEM SCANS ON LARGE REPOS

---

For FS scans:

* Scan only relevant directories

Example:

```
trivy fs ./services/api
```

Avoid:

```
trivy fs .
```

---

11. USE OFFLINE MODE IN STABLE ENVIRONMENTS

---

```
--offline-scan
```

Use when:

* DB is preloaded
* No outbound internet allowed

---

12. RECOMMENDED FAST CI COMMAND (PRODUCTION)

---

```
docker run --rm \
  -v trivy-cache:/root/.cache \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image \
  --severity HIGH,CRITICAL \
  --ignore-unfixed \
  --skip-db-update \
  --scanners vuln \
  --timeout 5m \
  myapp:latest
```

---

13. EXPECTED SCAN TIME (REALISTIC)

---

Without optimization:

* 2–4 minutes

With optimization:

* **10–30 seconds**
* Cached runs: **<10 seconds**

---

14. WHEN SCANS ARE STILL SLOW

---

Check:

* Image size (`docker images`)
* Network latency to registries
* CI runner CPU limits
* Docker-in-Docker overhead

---

15. NEXT LEVEL (OPTIONAL)

---

I can help you with:

* Ultra-fast Trivy pipelines (GitHub / GitLab / Jenkins)
* Trivy + SonarQube strategy (no duplication)
* DevSecOps gating policies
* Azure AKS hardened scanning model

If you want, tell me:
**CI tool + Docker/Kubernetes environment** and I will optimize it precisely.
==========================================================================================================