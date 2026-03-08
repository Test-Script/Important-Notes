=# Azure Kubernetes Service (AKS) Troubleshooting Guide

AKS is Microsoft's managed Kubernetes service, but like any complex system, issues can arise. This guide covers common troubleshooting scenarios and diagnostic approaches I've found useful in my work.

## Quick Diagnostic Tools

Before diving deep, use these built-in tools:

### AKS Diagnostics
Azure provides a comprehensive diagnostic experience in the portal:
1. Go to your AKS cluster in the Azure portal
2. Under "Support + troubleshooting", click "Diagnose and solve problems"
3. Run diagnostics for categories like:
   - Cluster connectivity
   - Node health
   - Networking issues
   - Security concerns

### kubectl Commands for Quick Checks
```bash
# Check cluster status
kubectl cluster-info

# View node status
kubectl get nodes

# Check pod health
kubectl get pods --all-namespaces

# View events for issues
kubectl get events --sort-by=.metadata.creationTimestamp
```

## Common Issues and Solutions

### 1. Cluster Not Accessible
**Symptoms:** Can't connect via kubectl, timeouts.

**Troubleshooting Steps:**
- Verify your kubeconfig: `kubectl config current-context`
- Check Azure CLI login: `az account show`
- Ensure correct subscription: `az account set --subscription <id>`
- Test network connectivity to API server

**Pro Tip:** In my experience, 90% of connectivity issues are authentication-related. Double-check your RBAC permissions.

### 2. Pods Stuck in Pending
**Symptoms:** Pods show "Pending" status indefinitely.

**Causes:**
- Insufficient resources (CPU/Memory)
- Node pool issues
- Storage problems

**Solutions:**
```bash
# Check pod details
kubectl describe pod <pod-name>

# View resource requests/limits
kubectl get pods -o jsonpath='{.spec.containers[*].resources}'

# Check node capacity
kubectl describe nodes
```

### 3. Networking Problems
**Symptoms:** Services not reachable, DNS issues.

**Debugging:**
- Use `kubectl exec` to test connectivity from within pods
- Check network policies if enabled
- Verify load balancer configuration
- Test with `nslookup` or `dig` for DNS issues

### 4. Node Issues
**Symptoms:** Nodes not ready, pods can't schedule.

**Checks:**
```bash
# Node status
kubectl get nodes -o wide

# Node details
kubectl describe node <node-name>

# System logs
kubectl logs -n kube-system <system-pod>
```

## Advanced Diagnostics

### Azure Monitor Integration
Enable Azure Monitor for containers for deeper insights:
- Real-time metrics
- Log aggregation
- Alerting on issues

### AKS Periscope
For complex issues, use AKS Periscope (a diagnostic tool):
```bash
# Run diagnostics
kubectl run aks-periscope --image=mcr.microsoft.com/aks/periscope --restart=Never -- /bin/bash -c "sleep 3600"
kubectl exec -it aks-periscope -- /bin/bash
```

## Best Practices from Experience

1. **Monitor Regularly:** Set up alerts for key metrics before issues occur.
2. **Use Namespaces:** Isolate workloads to prevent cascading failures.
3. **Keep Updated:** Regularly update AKS versions and node images.
4. **Test Upgrades:** Always test in staging before production upgrades.
5. **Document Changes:** Keep track of configuration changes for troubleshooting.

## Resources
- [AKS Troubleshooting Docs](https://docs.microsoft.com/en-us/azure/aks/troubleshoot)
- [AKS Best Practices](https://docs.microsoft.com/en-us/azure/aks/best-practices)
- Video: [How to troubleshoot common issues with AKS Diagnostics](https://www.youtube.com/watch?v=rZuC6m4WH10)

Remember, systematic troubleshooting saves time. Start with the basics and work your way up to advanced tools.