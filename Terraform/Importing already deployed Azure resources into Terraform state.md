🔷 Objective

Bring existing Azure resources under Terraform management without recreating them.

Terraform will:

    - NOT create the resource.
    - NOT modify it immediately.
    - Only attach it to the Terraform state.

🔷 High-Level Workflow

Step 1 → Write Terraform configuration for the resource
Step 2 → Run terraform init
Step 3 → Run terraform import
Step 4 → Run terraform plan (validate drift)
Step 5 → Adjust code if required

=======================================================================================
                                        Scenario
=======================================================================================

I am already having:

module "rg-identity" {
  source   = "./modules/resource_group"
  name     = var.rg_app_name
  location = var.location
}

Now I want to import:

rg-security-test  (already existing in Azure)

=======================================================================================
                                    Very Important Rule
=======================================================================================

If you want to manage multiple Resource Groups using same module, you must create another module block.

Terraform cannot import into something that does not exist in configuration.

=======================================================================================
                                    Modular + tfvars
=======================================================================================

Step 1 — Update terraform.tfvars

rg-identity = "rg-identity-test"
rg-security = "rg-security-test"
location            = "Central India"

tags = {
  Environment = "Development_Test"
}

Step 2 — Update root main.tf

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

Notice:

New module block

Different module name (rg-security)

Step 3 — Ensure Module Code Exists

resource "azurerm_resource_group" "this" {
  name     = var.name
  location = var.location
}

Step 4 — Initialize

terraform init

🔎 Step 5 — Get Azure Resource ID

az group show --name rg-shared-infra --query id -o tsv

Output :

/subscriptions/<sub-id>/resourceGroups/rg-shared-infra

🚀 Step 6 — Import (Modular Address)

Now the critical part.

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