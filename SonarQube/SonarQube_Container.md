![Image](https://miro.medium.com/v2/resize%3Afit%3A800/1%2Anb41Kfw3iwQwdW4FaatVrw.png)

![Image](https://miro.medium.com/1%2ACuZ9gJCp75AN4db8yButgg.jpeg)

![Image](https://miro.medium.com/1%2A5P9S62lNuasmSbdHrRAb_w.png)

![Image](https://europe1.discourse-cdn.com/sonarsource/uploads/sonarcommunity/original/2X/9/9930fd354a1f290d3c72518751aad43da3dd0206.png)

Below are **production-grade, step-by-step instructions** to run **SonarQube** using **Docker**, aligned with real-world DevOps practices.

---

## 1. Prerequisites

### System Requirements

```
OS            : Linux (Ubuntu/RHEL/CentOS preferred)
RAM           : Minimum 4 GB (8 GB recommended)
CPU           : 2 vCPU+
Disk          : 10–15 GB free
```

### Required Software

```
Docker        : >= 20.x
Docker Compose: >= v2
```

Verify:

```
docker --version
docker compose version
```

---

## 2. Mandatory Kernel Configuration (Very Important)

SonarQube uses Elasticsearch internally and **will not start** unless this is set.

```
sudo sysctl -w vm.max_map_count=262144
```

Persist after reboot:

```
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

Verify:

```
sysctl vm.max_map_count
```

---

## 3. Directory Structure (Recommended)

```
sonarqube-docker/
├── docker-compose.yml
├── data/
├── logs/
└── extensions/
```

Create directories:

```
mkdir -p sonarqube-docker/{data,logs,extensions}
cd sonarqube-docker
```

---

## 4. Docker Compose File (PostgreSQL + SonarQube)

Create `docker-compose.yml`:

```
version: "3.9"

services:
  postgres:
    image: postgres:15
    container_name: sonarqube-postgres
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonarqube
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  sonarqube:
    image: sonarqube:lts-community
    container_name: sonarqube
    depends_on:
      - postgres
    ports:
      - "9000:9000"
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://postgres:5432/sonarqube
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    volumes:
      - ./data:/opt/sonarqube/data
      - ./logs:/opt/sonarqube/logs
      - ./extensions:/opt/sonarqube/extensions
    restart: unless-stopped

volumes:
  postgres_data:
```

---

## 5. Start SonarQube

```
docker compose up -d
```

Check containers:

```
docker ps
```

---

## 6. Monitor Startup Logs (Critical for First Run)

```
docker logs -f sonarqube
```

Wait until you see:

```
SonarQube is up
```

Initial startup may take **2–5 minutes**.

---

## 7. Access SonarQube Web UI

```
URL      : http://<server-ip>:9000
Username : admin
Password : admin
```

You will be prompted to **change the password immediately**.

---

## 8. Common Issues & Fixes

### Issue: Web UI not responding

```
docker logs sonarqube
```

Check:

* `vm.max_map_count`
* Available RAM
* PostgreSQL container status

---

### Issue: Elasticsearch bootstrap error

```
ERROR: bootstrap checks failed
```

Fix:

```
sysctl vm.max_map_count
```

Must be `262144`.

---

### Issue: Permission denied on volumes

```
sudo chown -R 1000:1000 data logs extensions
```

Restart:

```
docker compose restart
```

---

## 9. Stop / Restart / Remove

### Stop

```
docker compose stop
```

### Restart

```
docker compose restart
```

### Remove everything (Clean Setup)

```
docker compose down -v
rm -rf data logs extensions
```

---

## 10. Recommended Production Enhancements

```
• Use NGINX reverse proxy (HTTPS)
• Use external managed PostgreSQL
• Enable backup for volumes
• Configure LDAP / SSO
• Add resource limits in docker-compose
```

---

## 11. Version Upgrade Strategy

```
1. Backup volumes
2. Update image tag
3. docker compose pull
4. docker compose up -d
```

---

If you want:

* **NGINX + SSL setup**
* **SonarQube + GitHub/GitLab CI**
* **Docker-only (without compose)**
* **Complete cleanup & reinstallation**
* **Security hardening**

state the requirement and environment, and I will provide a precise, production-ready procedure.
