# Terraform Credentials: Secure Authentication for Azure

Managing credentials in Terraform is like handling keys to your house—get it wrong, and security suffers. In my infrastructure-as-code journey, I've learned that proper credential management prevents breaches and simplifies multi-environment deployments. Let's explore secure ways to authenticate Terraform with Azure.

## Why Credential Management Matters

Terraform needs Azure access to create, update, and destroy resources. Poor credential handling leads to:
- **Security risks**: Secrets in version control
- **Maintenance headaches**: Hardcoded values everywhere
- **Team conflicts**: Different credentials per developer
- **Audit failures**: No credential rotation

## Authentication Methods

### 1. Azure CLI (Recommended for Development)

**Best for**: Local development, interactive sessions.

```bash
# Login interactively
az login

# Verify
az account show

# Terraform will use your CLI credentials automatically
terraform plan
```

**Pros**: No secrets stored, easy setup
**Cons**: Expires, not for CI/CD

### 2. Service Principal with Environment Variables

**Best for**: CI/CD pipelines, automated deployments.

Set environment variables:
```bash
export ARM_CLIENT_ID="your-app-id"
export ARM_CLIENT_SECRET="your-secret"
export ARM_SUBSCRIPTION_ID="your-sub-id"
export ARM_TENANT_ID="your-tenant-id"
```

**Security Note**: Never echo these in scripts or logs!

### 3. .env File Method (Project-Specific)

Perfect for managing multiple projects with different Service Principals.

**Step 1: Create .env file**
```bash
# .env file in project root
export ARM_CLIENT_ID="<appId>"
export ARM_CLIENT_SECRET="<password>"
export ARM_SUBSCRIPTION_ID="<subscriptionId>"
export ARM_TENANT_ID="<tenantId>"
```

**Step 2: Secure your repository**
Add to `.gitignore`:
```
# Local secret files
.env
*.tfvars
*.tfvars.json

# Terraform state (contains secrets!)
terraform.tfstate*
.terraform/
```

**Step 3: Workflow**
```bash
# Load credentials
source .env

# Verify (optional)
echo $ARM_CLIENT_ID  # Should show your ID

# Run Terraform
terraform plan
terraform apply
```

### 4. Managed Identity (Production-Grade)

**Best for**: Azure-hosted deployments (VMs, AKS, etc.)

No credentials needed—Azure handles authentication automatically.

```hcl
# In your Terraform config
provider "azurerm" {
  features {}
  use_msi = true
}
```

**Setup**: Enable managed identity on your deployment environment.

## Best Practices

### 1. Never Store Secrets in Code

**Wrong**:
```hcl
provider "azurerm" {
  client_id     = "hardcoded-secret"
  client_secret = "super-secret"
}
```

**Right**:
```hcl
provider "azurerm" {
  features {}
  # Uses environment variables automatically
}
```

### 2. Use Different SPs per Environment

- `dev-sp`: Contributor on dev subscription
- `prod-sp`: Reader + custom roles on prod
- `audit-sp`: Security Reader for compliance

### 3. Rotate Secrets Regularly

Set calendar reminders to:
- Generate new client secrets
- Update .env files
- Test deployments
- Revoke old secrets

### 4. Secure Local Development

- Use `.env` files (not committed)
- Consider `direnv` for automatic loading
- Use Azure CLI for personal development

## Troubleshooting Authentication Issues

**"Authentication failed"**:
- Check env vars: `env | grep ARM_`
- Verify SP exists: `az ad sp show --id $ARM_CLIENT_ID`
- Check permissions: `az role assignment list --assignee $ARM_CLIENT_ID`

**"Subscription not found"**:
- Verify subscription ID
- Ensure SP has access: `az account list`

**CI/CD pipeline fails**:
- Use secret variables in your CI system
- Avoid printing env vars in logs
- Test locally first

## Cross-References

- [Azure Service Principals](../Azure/Service%20Principle%20in%20Azure.md) for SP creation
- [Azure Pipeline Best Practices](../Azure/Azure%20Pipeline%20Best%20Practices.md) for CI/CD integration
- [Importing Resources](Importing%20already%20deployed%20Azure%20resources%20into%20Terraform%20state.md) for existing infrastructure

## Key Takeaways

1. **Environment variables over code**: Keep secrets out of version control
2. **.env files for projects**: Manage multiple environments cleanly
3. **Managed identity for production**: No secrets to manage
4. **Rotate regularly**: Treat secrets like passwords
5. **Test authentication**: Verify before running Terraform

Secure credential management is the foundation of trustworthy infrastructure. Start with Azure CLI for learning, then move to Service Principals for production. Your future self will thank you.