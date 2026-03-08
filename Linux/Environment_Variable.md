# Linux Environment Variables: The Invisible Configuration System

Environment variables are Linux's way of passing configuration to programs without hardcoding values. In my DevOps career, mastering env vars has saved countless hours of debugging. They're simple yet powerful—let's explore how they work and why they matter.

## Why Environment Variables Matter

Programs need configuration: database URLs, API keys, feature flags. Hardcoding these creates maintenance nightmares. Environment variables provide:
- **Runtime flexibility**: Change behavior without code changes
- **Security**: Keep secrets out of source code
- **Inheritance**: Child processes get parent variables
- **Standardization**: Universal across programming languages

## Variable Scopes: Local vs Global vs Permanent

Understanding scope prevents confusion. Let's break it down:

### Local Variables (Session-Only)

```bash
NAME="Gemini"
echo $NAME  # Works in this terminal
```

**Scope**: Only this specific terminal window. Gone when you close it.

### Exported Variables (Child Processes)

```bash
export NAME="Gemini"
echo $NAME  # Works here
bash       # Start child shell
echo $NAME  # Still works!
```

**Scope**: This terminal and any programs/scripts started from it.

### Permanent Variables (All Sessions)

Add to `~/.bashrc` or `~/.profile`:
```bash
echo 'export MY_VAR="persistent"' >> ~/.bashrc
source ~/.bashrc  # Reload
```

**Scope**: All future terminal sessions for that user.

## Testing Scope Yourself

Let's verify this behavior:

```bash
# Terminal 1
FOO="Local"
echo $FOO  # Shows: Local

# Start child shell
bash
echo $FOO  # Shows: (empty)
exit

# Back in parent
export FOO="Global"
bash
echo $FOO  # Shows: Global
```

**Key Insight**: `export` makes variables visible to child processes.

## Common Environment Variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `PATH` | Executable search paths | `/usr/bin:/bin` |
| `HOME` | User's home directory | `/home/user` |
| `USER` | Current username | `john` |
| `PWD` | Current working directory | `/home/user/projects` |
| `SHELL` | Default shell | `/bin/bash` |

## Kubernetes: Environment Variables at Scale

When moving to containers, env vars become even more critical. Kubernetes provides two mechanisms:

### ConfigMaps: Non-Sensitive Configuration

**Use for**: Public settings like database URLs, feature flags.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_HOST: "prod-db.example.com"
  LOG_LEVEL: "info"
```

**Injection into Pod**:
```yaml
envFrom:
- configMapRef:
    name: app-config
```

### Secrets: Sensitive Data

**Use for**: Passwords, API keys, certificates.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  DB_PASSWORD: "U3VwZXJTZWNyZXQxMjM="  # base64 encoded
```

**Injection**:
```yaml
envFrom:
- secretRef:
    name: app-secrets
```

## From Pod's Perspective

Your application code remains unchanged:
- Python: `os.getenv("DB_HOST")`
- Node.js: `process.env.DB_HOST`
- Java: `System.getenv("DB_HOST")`

Kubernetes acts as the "automated hand" setting variables before your program starts.

## Best Practices

1. **Use descriptive names**: `DATABASE_URL` not `DB`
2. **Validate required vars**: Check they exist at startup
3. **Document them**: List all required env vars in README
4. **Use defaults**: `VAR=${VAR:-default_value}`
5. **Secure secrets**: Never log or expose sensitive vars

## Troubleshooting

**Variable not set**:
```bash
echo $MY_VAR  # Check if exists
env | grep MY_VAR  # Search all vars
```

**Child process can't see var**:
- Did you `export` it?
- Check process tree with `pstree`

**Kubernetes injection fails**:
```bash
kubectl describe pod my-pod  # Check events
kubectl logs my-pod  # Check app logs
```

## Cross-References

- [Namespace and RBAC](../Kubernetes/Namespace,%20and%20RBAC.md) for K8s security
- [Azure Service Principals](../Azure/Service%20Principle%20in%20Azure.md) for cloud authentication
- [Terraform Credentials](../Terraform/Credentials_Invoking.md) for IaC secrets

## Key Takeaways

1. **Scope matters**: Local vs exported vs permanent
2. **Export for inheritance**: Child processes need `export`
3. **Kubernetes abstraction**: ConfigMaps/Secrets replace manual setup
4. **Security first**: Use Secrets for sensitive data
5. **Test thoroughly**: Verify vars reach your application

Environment variables are the duct tape of Linux systems—simple, effective, and essential. Master them, and your deployments become much more manageable.