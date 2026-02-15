C:\Users\DELL\Documents\Terraform_Project\Azurerm_Terraform\env\dev>terraform plan module.rg.azurerm_resource_group.this: Refreshing state... [id=/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-identity-test] module.rg-security.azurerm_resource_group.this: Refreshing state... [id=/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-security-test] Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols: + create - destroy Terraform will perform the following actions: # module.rg.azurerm_resource_group.this will be destroyed # (because azurerm_resource_group.this is not in configuration) - resource "azurerm_resource_group" "this" { - id = "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-identity-test" -> null - location = "centralindia" -> null - name = "rg-identity-test" -> null - tags = { - "Environment" = "Development_Test" } -> null # (1 unchanged attribute hidden) } # module.rg-identity.azurerm_resource_group.this will be created + resource "azurerm_resource_group" "this" { + id = (known after apply) + location = "centralindia" + name = "rg-identity-test" + tags = { + "Environment" = "Development_Test" } } Plan: 1 to add, 0 to change, 1 to destroy. Changes to Outputs: - resource_group_id = "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-identity-test" -> null - resource_group_name = "rg-identity-test" -> null

=======================================================================================
                                state address mismatch issue
=======================================================================================

🔎 What Happened

Earlier my module was:

    module "rg"

So state contains:

    module.rg.azurerm_resource_group.this

Now I have renamed it to:

    module "rg-identity"

So Terraform sees:

    module.rg-identity.azurerm_resource_group.this

These are two different state addresses.

Terraform thinks:

Old one → must destroy

New one → must create

Even though it is the SAME Azure Resource Group.

🚫 DO NOT RUN APPLY

If you apply, Terraform will:

Try to delete rg-identity-test

Then recreate it

That is not what you want.

✅ Correct Fix → Move State

You must tell Terraform:

"This resource moved from one module path to another."

Use:

terraform state mv \
module.rg.azurerm_resource_group.this \
module.rg-identity.azurerm_resource_group.this

🔍 What This Does

It updates only the state file.

No Azure changes.
No resource recreation.

🔄 After That

Run:

terraform plan

Expected output:

No changes. Infrastructure is up-to-date.

🧠 Why This Happens (Core Concept)

Terraform tracks resources by:

<module path> + <resource type> + <resource name>

NOT by Azure ID.

So even if Azure ID is same, Terraform sees:

module.rg
≠
module.rg-identity

Hence destroy + create.

=======================================================================================
                                Golden Rule
=======================================================================================

If you:

Rename module

Rename resource block

Move resource into another module

We must use:

terraform state mv

Otherwise Terraform will destroy and recreate.

=======================================================================================
                            🏗 Enterprise Recommendation
=======================================================================================

When refactoring module names in production:

Always:

    terraform state list
    terraform state mv
    terraform plan

Never rename blindly.
