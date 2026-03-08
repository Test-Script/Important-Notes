# Azure DevOps Pipelines: Best Practices for Terraform Deployments

Azure Pipelines with Terraform can be a game-changer for infrastructure as code, but getting it right requires attention to security, reliability, and maintainability. In my experience managing CI/CD for multiple teams, these practices have prevented countless production issues. Let's build robust pipelines that scale.

## Why Pipeline Best Practices Matter

Poorly configured pipelines lead to:
- **Security breaches**: Exposed secrets in logs
- **Failed deployments**: Race conditions and resource conflicts
- **Maintenance nightmares**: Hardcoded values everywhere
- **Audit failures**: No approval gates for production

## Secure Credential Management

Never store secrets in code. Use Azure DevOps variable groups.

### Environment Variables (Simple Setup)

Set in pipeline variables:
```
ARM_CLIENT_ID        = <clientId>
ARM_CLIENT_SECRET    = <clientSecret>   (mark as secret)
ARM_TENANT_ID        = <tenantId>
ARM_SUBSCRIPTION_ID  = <subscriptionId>
```

**Warning**: Mark ARM_CLIENT_SECRET as secret to prevent logging.

### Variable Groups (Recommended for Teams)

Create reusable credential sets:

1. Go to Pipelines → Library → Variable Groups → + New
2. Add variables with ARM_ prefixes
3. Mark secrets appropriately
4. Name it descriptively (e.g., "Terraform-Prod-Credentials")

## Sample Pipeline Configuration

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

variables:
- group: Terraform-Variables  # Reference your variable group

stages:
- stage: Validate
  jobs:
  - job: TerraformValidate
    steps:
    - task: TerraformInstaller@1
      inputs:
        terraformVersion: '1.7.5'

    - script: |
        echo "Initializing Terraform..."
        terraform init
      displayName: Terraform Init
      env:
        ARM_CLIENT_ID: $(ARM_CLIENT_ID)
        ARM_CLIENT_SECRET: $(ARM_CLIENT_SECRET)
        ARM_TENANT_ID: $(ARM_TENANT_ID)
        ARM_SUBSCRIPTION_ID: $(ARM_SUBSCRIPTION_ID)

    - script: |
        terraform validate
      displayName: Terraform Validate

- stage: Plan
  jobs:
  - job: TerraformPlan
    steps:
    - task: TerraformInstaller@1
      inputs:
        terraformVersion: '1.7.5'

    - script: |
        terraform init
      displayName: Terraform Init
      env:
        ARM_CLIENT_ID: $(ARM_CLIENT_ID)
        ARM_CLIENT_SECRET: $(ARM_CLIENT_SECRET)
        ARM_TENANT_ID: $(ARM_TENANT_ID)
        ARM_SUBSCRIPTION_ID: $(ARM_SUBSCRIPTION_ID)

    - script: |
        terraform plan -out=tfplan
      displayName: Terraform Plan
      env:
        ARM_CLIENT_ID: $(ARM_CLIENT_ID)
        ARM_CLIENT_SECRET: $(ARM_CLIENT_SECRET)
        ARM_TENANT_ID: $(ARM_TENANT_ID)
        ARM_SUBSCRIPTION_ID: $(ARM_SUBSCRIPTION_ID)

    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: 'tfplan'
        artifact: 'tfplan'

- stage: Deploy
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: TerraformApply
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: DownloadPipelineArtifact@2
            inputs:
              artifact: 'tfplan'

          - script: |
              terraform apply -auto-approve tfplan
            displayName: Terraform Apply
            env:
              ARM_CLIENT_ID: $(ARM_CLIENT_ID)
              ARM_CLIENT_SECRET: $(ARM_CLIENT_SECRET)
              ARM_TENANT_ID: $(ARM_TENANT_ID)
              ARM_SUBSCRIPTION_ID: $(ARM_SUBSCRIPTION_ID)
```

## Terraform Provider Configuration

Keep your Terraform code clean:

```hcl
# provider.tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
  # Uses ARM_* environment variables automatically
}
```

**Pro Tip**: No need to define variables if using standard ARM_ env vars.

## Advanced: Approval Gates

Add production safeguards:

1. Create environment: Pipelines → Environments → + New
2. Set "Production" checks requiring approval
3. Reference in deployment job: `environment: 'production'`

## Best Practices

### 1. Use Stages for Separation

- **Validate**: Syntax and security checks
- **Plan**: Generate execution plan
- **Deploy**: Apply with approvals

### 2. Secure Secrets

- Use variable groups, not pipeline variables
- Rotate credentials regularly
- Use Azure Key Vault for sensitive data

### 3. State Management

```yaml
- script: |
    terraform init -backend-config="storage_account_name=$(TF_STATE_STORAGE)"
  displayName: Init with Remote State
```

### 4. Error Handling

```yaml
- script: |
    set -e
    terraform plan -detailed-exitcode
  displayName: Plan with Exit Codes
  continueOnError: true
```

### 5. Notifications

Add email/webhook notifications for failures.

## Troubleshooting Pipeline Issues

**"Authentication failed"**:
- Verify variable group access
- Check secret marking
- Test locally with same env vars

**"State lock"**:
- Use `terraform force-unlock` carefully
- Check for stuck agents

**"Resource conflicts"**:
- Add dependencies between jobs
- Use locks for shared resources

## Cross-References

- [Terraform Credentials](../Terraform/Credentials_Invoking.md) for local setup
- [Azure Service Principals](Service%20Principle%20in%20Azure.md) for authentication
- [Azure DevOps Agent Insights](Azure%20DevOps%20Agent%20Insights.md) for agent configuration

## Key Takeaways

1. **Secure first**: Use variable groups and mark secrets
2. **Stage your deployments**: Validate → Plan → Deploy
3. **Add approvals**: Never auto-deploy to production
4. **Remote state**: Essential for team collaboration
5. **Monitor and alert**: Catch issues early

Well-configured pipelines turn infrastructure changes from risky events into routine operations. Start with the basics, then layer on approvals and monitoring as you scale.

You cannot put an approval inside steps.
Approval must be applied at the stage level, using an Environment that has an approval configured.

========================= Create an Azure DevOps Environment with Approval

In Azure DevOps:

Environments → New Environment → Name: prod-approval

Inside the environment:

Approvals and Checks → Add → Approval → Add approvers.

Save.

========================= Use Stages in YAML and Place Apply in a Separate Stage

trigger:
- main

pool:
  name: Self_Agent_Pool

variables:
- group: Terraform-Variables

stages:

# ---------------------------
# Stage 1: Init + Plan
# ---------------------------
- stage: TerraformPlan
  displayName: "Terraform Initialization & Planning"
  jobs:
  - job: plan
    displayName: "Terraform Plan Job"
    steps:
    - task: TerraformInstaller@1
      inputs:
        terraformVersion: '1.7.5'

    - script: |
        echo "Initializing Terraform..."
        terraform init
      displayName: Terraform Init
      env:
        ARM_CLIENT_ID: $(ARM_CLIENT_ID)
        ARM_CLIENT_SECRET: $(ARM_CLIENT_SECRET)
        ARM_TENANT_ID: $(ARM_TENANT_ID)
        ARM_SUBSCRIPTION_ID: $(ARM_SUBSCRIPTION_ID)

    - script: |
        terraform plan -out=tfplan
      displayName: Terraform Plan
      env:
        ARM_CLIENT_ID: $(ARM_CLIENT_ID)
        ARM_CLIENT_SECRET: $(ARM_CLIENT_SECRET)
        ARM_TENANT_ID: $(ARM_TENANT_ID)
        ARM_SUBSCRIPTION_ID: $(ARM_SUBSCRIPTION_ID)

    - publish: tfplan
      artifact: planfile


# ---------------------------
# Stage 2: Apply (Approval Required)
# ---------------------------
- stage: TerraformApply
  displayName: "Terraform Apply Stage"
  dependsOn: TerraformPlan
  condition: succeeded()  
  environment: prod-approval     # Triggers approval gate
  jobs:
  - job: apply
    displayName: "Terraform Apply Job"
    steps:

    - script: |
        echo "Downloading plan file..."
      displayName: "Prepare Apply"

    - download: current
      artifact: planfile

    - script: |
        terraform apply -auto-approve tfplan
      displayName: Terraform Apply
      env:
        ARM_CLIENT_ID: $(ARM_CLIENT_ID)
        ARM_CLIENT_SECRET: $(ARM_CLIENT_SECRET)
        ARM_TENANT_ID: $(ARM_TENANT_ID)
        ARM_SUBSCRIPTION_ID: $(ARM_SUBSCRIPTION_ID)
		
===================== How It Works

Stage 1 (TerraformPlan) runs init + plan.

It publishes tfplan as an artifact.

Stage 2 (TerraformApply) references an Environment (prod-approval).

Since that Environment has approval enabled, Azure DevOps pauses the pipeline before entering this stage.

After manual approval is given, Apply proceeds.

==================================================== Azure Terraform =====================================================

…or create a new repository on the command line

echo "# Terraforn_Repo" >> README.md
git init
git add README.md
git commit -m "Newelly Created Terraform Pipeline"
git branch -M main
git remote add origin https://github.com/Test-Script/Terraforn_Repo.git
git push -u origin main

…or push an existing repository from the command line

git remote add origin https://github.com/Test-Script/Terraforn_Repo.git
git branch -M main
git push -u origin main

============================================== Git Hub Best Practices ====================================================

# Recommendation: which files to ignore (and why)

Below is a concise, copy/paste-friendly `.gitignore` you can use for Terraform projects plus explicit explanations and tidy git commands to remove sensitive files that may already be committed. Follow these steps immediately — state files and provider binaries frequently contain secrets or sensitive metadata and **must not** be stored in a public or shared repo.

==================================

# `.gitignore` (Notepad++ / plain text)

```
# Local terraform state and backups (contains secrets, resource IDs, outputs)
terraform.tfstate
terraform.tfstate.backup

# Local crash logs / plan files
crash.log
*.tfplan

# CLI local folder + downloaded providers (do NOT commit provider binaries or .terraform)
.terraform/
.terraform.lock.hcl   # keep? see note below

# Variable files that commonly contain secrets
*.tfvars
*.tfvars.json
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Local credentials / SDK auth files you might create
*.credentials.json
azureCredentials.json
sp.json
tf-demo-sp-credentials.json

# OS / editor files
.vscode/
.idea/
.DS_Store
Thumbs.db

# SSH private keys (never commit)
id_rsa
id_rsa.pub
*.pem
*.key
```

==================================

# Important notes (short & specific)

* **Do ignore** `terraform.tfstate` and `terraform.tfstate.backup`. These files contain resource IDs, IPs, and often sensitive outputs (passwords, secrets). Always store state in a remote backend (Azure Storage, Terraform Cloud, S3 + DynamoDB, etc.) with access control and encryption.
* **Do ignore** the entire `.terraform/` directory. It contains provider binaries and cached modules — not source code.
* **Commit** `.terraform.lock.hcl` normally: it pins provider dependency checksums and **should** be committed to ensure deterministic provider versions. (If you have a policy to vendor provider binaries, do **not** commit provider executables; keep `.terraform/` ignored.)
* **Do ignore** `*.tfvars` or any file that contains secrets (client secrets, passwords). Use environment variables or a secret manager in CI.
* **Never commit** SSH private keys, client secret JSON files (like `tf-demo-sp-credentials.json`), or any file produced by `az ad sp create-for-rbac --sdk-auth`.

---

# If sensitive files are already committed — remove them from Git (safe sequence)

1. Add `.gitignore` (create file with content above)
2. Remove sensitive files from the index but keep them locally:

==================================
git rm --cached terraform.tfstate terraform.tfstate.backup
git rm --cached -r .terraform
git rm --cached *.tfvars *.tfvars.json tf-demo-sp-credentials.json sp.json azureCredentials.json || true
git add .gitignore
git commit -m "remove sensitive Terraform files from repo and add .gitignore"
==================================

3. If secrets were committed in earlier commits you must **purge history** (and then rotate secrets):

   * For a simple purge of specific files you can use the BFG or `git filter-branch` / `git filter-repo`. Example with BFG (recommended for ease):

==================================
# Download and run BFG (external tool) - example steps (install BFG separately)
bfg --delete-files tf-demo-sp-credentials.json
bfg --delete-files terraform.tfstate
# then
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force
==================================

* If you cannot run BFG, use `git filter-repo` or `git filter-branch` — but be careful: rewriting history requires force-push and coordination with other collaborators.

4. **Rotate any secrets** that were exposed (client secrets, service principal secrets, SSH keys) immediately — assume leakage once committed, even if you subsequently deleted from history.

---

# Secure alternatives & best practices (practical checklist)

* Use a **remote backend** for Terraform state (Azure Blob Storage with access restricted by RBAC + storage account key or managed identity). This prevents local `terraform.tfstate` files and enables state locking.
* Use **environment variables** for provider credentials (e.g., `ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_SUBSCRIPTION_ID`, `ARM_TENANT_ID`) or a CI secret store; avoid baking secrets into `.tf` files.
* Use **Azure Key Vault** / secret manager for secrets that your Terraform code needs to reference at apply time.
* Use **least privilege** for Service Principals (scope them to resource groups and use minimal roles or custom roles).
* Add a pre-commit hook or `git-secrets`/`gitleaks` scanner to prevent accidental commits of high-risk patterns (AWS keys, Azure client secrets, private keys).
* Limit file permissions for private keys: on Windows use `icacls`, on Unix use `chmod 600 ~/.ssh/id_rsa`.

======================== Working with push protection from the command line =============================================

Removing a secret introduced by an earlier commit on your branch

Examine the error message that displayed when you tried to push your branch, which lists all of the commits that contain the secret.

$ git push -u origin main
Enumerating objects: 11, done.
Counting objects: 100% (11/11), done.
Delta compression using up to 8 threads
Compressing objects: 100% (10/10), done.
Writing objects: 100% (11/11), 1.76 KiB | 900.00 KiB/s, done.
Total 11 (delta 3), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (3/3), done.
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote:
remote: - GITHUB PUSH PROTECTION
remote:   —————————————————————————————————————————
remote:     Resolve the following violations before pushing again
remote:
remote:     - Push cannot contain secrets
remote:
remote:
remote:      (?) Learn how to resolve a blocked push
remote:      https://docs.github.com/code-security/secret-scanning/working-with-secret-scanning-and-push-protection/working-with-push-protection-from-the-command-line#resolving-a-blocked-push
remote:
remote:
remote:       —— Azure Active Directory Application Secret —————————
remote:        locations:
remote:          - commit: d9bfe3769e2f3d7d9b02f06ebc0dfe9d52776470
remote:            path: provider.tf:16
remote:
remote:        (?) To push, remove secret from commit(s) or follow this URL to allow the secret.
remote:        https://github.com/Test-Script/Terraforn_Repo/security/secret-scanning/unblock-secret/36l1dhiWgJMMMj5wy6oJgXnxY0R
remote:
remote:
remote:
To https://github.com/Test-Script/Terraforn_Repo.git
 ! [remote rejected] main -> main (push declined due to repository rule violations)
error: failed to push some refs to 'https://github.com/Test-Script/Terraforn_Repo.git'

==============================

Next, run git log to see a full history of all the commits on your branch, along with their corresponding timestamps.

DELL@Tejas MINGW64 ~/Documents/Terraform (main)
$ git log --oneline --decorate --graph -n 50
* 0dbd43b (HEAD -> main) remove sensitive Terraform files from repo and add .gitignore
* d9bfe37 remove sensitive Terraform files from repo and add .gitignore

===========

DELL@Tejas MINGW64 ~/Documents/Terraform (main)
$ git log
commit 0dbd43ba765701f4724d7a59a409802b0830ae44 (HEAD -> main)
Author: Tejas Targhale <tejastarghale47@gmail.com>
Date:   Sat Dec 13 00:49:47 2025 +0530

    remove sensitive Terraform files from repo and add .gitignore

commit d9bfe3769e2f3d7d9b02f06ebc0dfe9d52776470
Author: Tejas Targhale <tejastarghale47@gmail.com>
Date:   Sat Dec 13 00:36:39 2025 +0530

    remove sensitive Terraform files from repo and add .gitignore

=================================

Focusing only on the commits that contain the secret, use the output of git log to identify which commit comes earliest in your Git history.

In the example, commit d9bfe3769e2f3d7d9b02f06ebc0dfe9d52776470 was the first commit to contain the secret.

=================================

Start an interactive rebase with git rebase -i <COMMIT-ID>~1.

git rebase -i 0925265~1

For <COMMIT-ID>, use the commit identified in step 3. For example, git rebase -i d9bfe3769e2f3d7d9b02f06ebc0dfe9d52776470~1.

=================================

In the editor, choose to edit the commit identified in step 3 by changing pick to edit on the first line of the text.

commit 9800c5fa966f7700c10eb140278c4aa173b7aa6d
Author: Tejas Targhale <tejastarghale47@gmail.com>
Date:   Sat Dec 13 12:51:06 2025 +0530

========================================== Git Conflict Resolve for Sensitive Information Exclusion ======================

Below is the **exact root cause and the only correct fix**, written in **Notepad++ format**, to permanently resolve this **GH013 secret scanning block**.

==================================================
WHY THIS IS STILL FAILING (KEY POINT)
=====================================

Even though you **deleted `Terraform Setup.txt`**, GitHub is STILL blocking the push because:

✔ The secret **exists in Git history**
✔ Specifically in commit: `a6ff9ae2d8551ee1aa9d7df53af0248d079db036`
✔ GitHub scans **ALL commits being pushed**, not just current files

Deleting the file alone is **NOT sufficient**.

==================================================
CONFIRMED ROOT CAUSE
====================

* `Terraform Setup.txt`
* Line 12
* Contains **Azure AD Application (Client) Secret**
* Present in at least **one historical commit**

GitHub Push Protection will block **every push** until that commit is rewritten.

==================================================
THE ONLY CORRECT SOLUTION (NO SHORTCUTS)
========================================

You must **rewrite Git history** and **completely remove the file from ALL commits**.

This is mandatory.

==================================================
STEP 0: ROTATE THE AZURE SECRET (MANDATORY)
===========================================

Before continuing:

✔ Go to Azure Portal
✔ Delete the exposed Client Secret
✔ Create a NEW secret
✔ Use it ONLY via:

* Azure DevOps Variable Group (Secret)
* GitHub Secrets
* Azure Key Vault
* Environment variables

==================================================
STEP 1: ABORT ANY ONGOING REBASE (IF ACTIVE)
============================================

Run:

```
git rebase --abort
```

This ensures a clean state before history rewrite.

==================================================
STEP 2: REMOVE THE FILE FROM ENTIRE GIT HISTORY
===============================================

Run **from repo root**:

```
git filter-repo --path "Terraform Setup.txt" --invert-paths
```

What this does:
✔ Deletes the file from **all commits**
✔ Removes the secret completely
✔ Makes GitHub happy

NOTE:

* This will remove **all remotes** automatically (expected behavior)

==================================================
STEP 3: RE-ADD THE REMOTE
=========================

```
git remote add origin https://github.com/Test-Script/Terraforn_Repo.git
```

Verify:

```
git remote -v
```

==================================================
STEP 4: FIX .gitignore (VERY IMPORTANT)
=======================================

Ensure `.gitignore` contains:

```
# Terraform
.terraform/
terraform.tfstate
terraform.tfstate.backup
Terraform Setup.txt
*.tfvars
*.tfvars.json
```

Then commit `.gitignore` if not already committed:

```
git add .gitignore
git commit -m "Harden gitignore for Terraform secrets and state"
```

==================================================
STEP 5: FORCE PUSH CLEAN HISTORY
================================

This step is REQUIRED because history was rewritten:

```
git push origin main --force
```

==================================================
STEP 6: VERIFY SECRET IS GONE (OPTIONAL BUT RECOMMENDED)
========================================================

Run:

```
git log --all -- "Terraform Setup.txt"
```

Expected:

```
(no output)
```

==================================================
DO NOT DO THESE (IMPORTANT)
===========================

✘ Do NOT click the GitHub “Unblock Secret” link
✘ Do NOT commit secrets again
✘ Do NOT store credentials in `.txt` or `.tfvars`
✘ Do NOT rely on deleting files without rewriting history

==================================================
CORRECT WAY TO HANDLE AZURE CREDENTIALS
=======================================

BEST OPTIONS (IN ORDER):

1. **Managed Identity** (no secrets at all)
2. Azure DevOps → Variable Group (secret)
3. GitHub → Repository Secrets
4. Azure Key Vault + Terraform provider

Example (recommended):

```
provider "azurerm" {
  features {}
}
```

(no client secret required when using Managed Identity)

==================================================
FINAL STATE (SUCCESS CRITERIA)
==============================

✔ No secrets in Git history
✔ Push protection passes
✔ `.terraform/` ignored
✔ `terraform.tfstate` ignored
✔ Clean, professional Terraform repo

==================================================
SUMMARY (ONE LINE)
==================

Deleting the file was NOT enough — you must **rewrite Git history using `git filter-repo` and force push**.


