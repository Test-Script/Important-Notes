# Importing Existing Azure Resources into Terraform State

One of the most powerful Terraform features is importing existing infrastructure. In my infrastructure migrations, this has saved weeks of work by bringing manually created resources under Terraform management. Instead of recreating everything, you can adopt existing resources safely. Let's explore how to do this effectively.

## Why Import Matters

**The Problem**: You have Azure resources created manually or by other tools, but now want Terraform to manage them.

**The Solution**: Import brings resources into Terraform state without recreating them. Terraform learns about the existing resource and can manage it going forward.

**Key Benefits**:
- **No downtime**: Resources stay running
- **Gradual migration**: Import in phases
- **State consistency**: Everything tracked in Terraform

## High-Level Import Workflow

1. **Write configuration** for the resource in Terraform
2. **Initialize** Terraform in the directory
3. **Import** the resource into state
4. **Plan** to check for configuration drift
5. **Adjust** code to match actual resource

## Practical Example: Importing Resource Groups

### Scenario Setup

You have existing resource groups:
- `rg-identity-test` (already in Terraform)
- `rg-security-test` (needs importing)

Current Terraform code:
```hcl
module "rg-identity" {
  source   = "./modules/resource_group"
  name     = var.rg_app_name
  location = var.location
}
```

### Step 1: Update Variables

Add the new resource group to your `terraform.tfvars`:
```hcl
rg-identity = "rg-identity-test"
rg-security = "rg-security-test"
location    = "Central India"

tags = {
  Environment = "Development_Test"
}
```

### Step 2: Add Module Block

Create a new module instance for the resource to import:
```hcl
module "rg-identity" {
  source   = "../../modules/resource_group"
  name     = var.rg-identity
  location = var.location
  tags     = var.tags
}

module "rg-security" {
  source   = "../../modules/resource_group"
  name     = var.rg-security
  location = var.location
}
```

**Important**: Each resource needs its own module block. Terraform can't import into non-existent configuration.

### Step 3: Verify Module Code

Ensure your module creates the right resource:
```hcl
# modules/resource_group/main.tf
resource "azurerm_resource_group" "this" {
  name     = var.name
  location = var.location
  tags     = var.tags
}
```

### Step 4: Initialize Terraform

```bash
terraform init
```

### Step 5: Get Azure Resource ID

Find the resource ID for importing:
```bash
az group show --name rg-security-test --query id -o tsv
```

**Output**: `/subscriptions/<sub-id>/resourceGroups/rg-security-test`

### Step 6: Import the Resource

Use the module address for import:
```bash
terraform import module.rg-security.azurerm_resource_group.this /subscriptions/<sub-id>/resourceGroups/rg-security-test
```

**Module Address Format**: `module.<module_name>.<resource_type>.<resource_name>`

### Step 7: Verify Import

Check that the resource is now in state:
```bash
terraform state list
terraform show module.rg-security
```

### Step 8: Plan and Adjust

Run plan to check for configuration differences:
```bash
terraform plan
```

**Common Issues**:
- **Drift detected**: Adjust your configuration to match Azure
- **Missing attributes**: Add tags, location, etc. to match

## Advanced Import Scenarios

### Importing Multiple Resources

```bash
# Import VM
terraform import module.vm.azurerm_virtual_machine.this /subscriptions/.../resourceGroups/.../providers/Microsoft.Compute/virtualMachines/myvm

# Import network
terraform import module.network.azurerm_virtual_network.this /subscriptions/.../resourceGroups/.../providers/Microsoft.Network/virtualNetworks/myvnet
```

### Using Import Blocks (Terraform 1.5+)

```hcl
import {
  to = azurerm_resource_group.example
  id = "/subscriptions/.../resourceGroups/example"
}

resource "azurerm_resource_group" "example" {
  name     = "example"
  location = "West Europe"
}
```

### Bulk Import with Scripts

For many resources, create a script:
```bash
#!/bin/bash
# Get all resource groups
az group list --query '[].{name:name, id:id}' -o tsv | while read name id; do
  echo "Importing $name..."
  terraform import "azurerm_resource_group.$name" "$id"
done
```

## Best Practices

### 1. Plan Your Imports
- **Group by environment**: Import dev, then staging, then prod
- **Test in non-prod**: Validate import process first
- **Document dependencies**: Import in dependency order

### 2. Handle Configuration Drift
- **Review plan output**: Understand differences
- **Preserve important settings**: Don't overwrite critical configs
- **Use data sources**: For read-only references

### 3. State Management
- **Backup state**: Before bulk imports
- **Use remote state**: For team collaboration
- **Lock state**: Prevent concurrent modifications

## Troubleshooting Import Issues

**"Resource already managed"**:
- Check existing state: `terraform state list`
- Remove from state if needed: `terraform state rm`

**"Import target not found"**:
- Verify resource exists in Azure
- Check resource ID format
- Ensure correct module address

**"Configuration doesn't match"**:
- Run `terraform show` to see imported config
- Adjust your code to match actual resource
- Use `terraform taint` if needed

## Cross-References

- [Terraform Credentials](Credentials_Invoking.md) for authentication setup
- [Azure Service Principals](../Azure/Service%20Principle%20in%20Azure.md) for Azure access
- [Azure Pipeline Best Practices](../Azure/Azure%20Pipeline%20Best%20Practices.md) for CI/CD integration

## Key Takeaways

1. **Import doesn't recreate**: Resources stay running
2. **Configuration first**: Write Terraform code before importing
3. **Module addresses matter**: Use correct paths for modular imports
4. **Plan after import**: Always check for configuration drift
5. **Start small**: Test with one resource before bulk imports

Importing transforms existing infrastructure into manageable code. It's like adopting existing pets rather than buying new ones—more work initially, but worth it for long-term control. Start with simple resources and build confidence with complex ones.

Our resource is:

module.rg_shared.azurerm_resource_group.this

Import command:

terraform import module.rg-security.azurerm_resource_group.this /subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-identity-test

🔍 Step 7 — Validate

terraform plan

No changes. Infrastructure is up-to-date.

Step 8 - Confirm State

terraform state list

You should see:

module.rg_app.azurerm_resource_group.this
module.rg_shared.azurerm_resource_group.this


=======================================================================================
                        Enterprise-Level Clean Pattern (Better Design)
=======================================================================================

Instead of multiple module blocks, you can design like this:

variable "resource_groups" {
  type = map(string)
}

terraform.tfvars:

resource_groups = {
  app    = "rg-app-prod"
  shared = "rg-shared-infra"
}

root main.tf:

module "rg" {
  source   = "./modules/resource_group"
  for_each = var.resource_groups

  name     = each.value
  location = var.location
}

Then import like this:

terraform import \
'module.rg["shared"].azurerm_resource_group.this' \
/subscriptions/<sub-id>/resourceGroups/rg-shared-infra

This is the scalable pattern.

=======================================================================================                             🧠 Final Mental Model
=======================================================================================
Import works only if:

Resource block exists
↓
Correct module path
↓
Correct Azure ID
↓
terraform plan shows zero drift

======================================================================================
Example Of Execution I have been added here
======================================================================================


C:\Users\DELL\Documents\Terraform_Project\Azurerm_Terraform\env\dev>az group show --name rg-vm-test --query id -o tsv
/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test

C:\Users\DELL\Documents\Terraform_Project\Azurerm_Terraform\env\dev>terraform import module.rg-vm.azurerm_resource_group.this /subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test
╷
│ Error: Module not installed
│
│   on main.tf line 20:
│   20: module "rg-vm" {
│
│ This module is not yet installed. Run "terraform init" to install all modules required by this configuration.
╵


C:\Users\DELL\Documents\Terraform_Project\Azurerm_Terraform\env\dev>terraform init
Initializing the backend...
Initializing modules...
- rg-vm in ..\..\modules\resource_group
Initializing provider plugins...
- Reusing previous version of hashicorp/azurerm from the dependency lock file
- Using previously-installed hashicorp/azurerm v3.117.1

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

C:\Users\DELL\Documents\Terraform_Project\Azurerm_Terraform\env\dev>terraform import module.rg-vm.azurerm_resource_group.this /subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test
module.rg-vm.azurerm_resource_group.this: Importing from ID "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test"...
module.rg-vm.azurerm_resource_group.this: Import prepared!
  Prepared azurerm_resource_group for import
module.rg-vm.azurerm_resource_group.this: Refreshing state... [id=/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.


C:\Users\DELL\Documents\Terraform_Project\Azurerm_Terraform\env\dev>terraform plan
module.rg-identity.azurerm_resource_group.this: Refreshing state... [id=/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-identity-test]
module.rg-security.azurerm_resource_group.this: Refreshing state... [id=/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-security-test]
module.rg-network-security.azurerm_resource_group.this: Refreshing state... [id=/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test]
module.rg-vm.azurerm_resource_group.this: Refreshing state... [id=/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

C:\Users\DELL\Documents\Terraform_Project\Azurerm_Terraform\env\dev>terraform state list
module.rg-identity.azurerm_resource_group.this
module.rg-network-security.azurerm_resource_group.this
module.rg-security.azurerm_resource_group.this
module.rg-vm.azurerm_resource_group.this

C:\Users\DELL\Documents\Terraform_Project\Azurerm_Terraform\env\dev>

===========================================================================================