Gemini said
Using a .env file is a great way to manage multiple projects with different Service Principals without cluttering your global system variables.

Here is the step-by-step breakdown of how to set this up securely and efficiently.

1. Create the .env File
In your Terraform project root, create a file named .env. Note that we use export so the variables are available to the Terraform process when you run the command.

Bash
# .env file
export ARM_CLIENT_ID="<appId>"
export ARM_CLIENT_SECRET="<password>"
export ARM_SUBSCRIPTION_ID="<subscriptionId>"
export ARM_TENANT_ID="<tenantId>"
2. Secure Your Repository
Before you do anything else, ensure you don't accidentally push these secrets to a public or private repository.

Add this to your .gitignore:

Plaintext
# Local secret files
.env
*.tfvars
*.tfvars.json

# Terraform state (contains secrets in plain text!)
terraform.tfstate*
.terraform/
3. The Workflow
When you open a new terminal to start working, you need to "load" these variables into your current session.

Load the secrets: source .env

Verify (Optional): echo $ARM_CLIENT_ID (to make sure it's there)

Run Terraform: terraform plan


# Local secret files
.env
*.tfvars
*.tfvars.json

# Terraform state (contains secrets in plain text!)
terraform.tfstate*
.terraform/