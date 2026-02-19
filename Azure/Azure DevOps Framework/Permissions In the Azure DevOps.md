============================================================
**AZURE DEVOPS – GROUP PERMISSIONS**
============================================================

Azure DevOps uses a **Role-Based Access Control (RBAC)** model.
Permissions are assigned at different scopes:

✔ Organization Level
✔ Project Level
✔ Repo Level
✔ Pipeline Level
✔ Environment Level
✔ Agent Pool Level

Below is the complete breakdown.

============================================================
**1. ORGANIZATION-LEVEL GROUPS & PERMISSIONS**
==============================================

## **1.1. Project Collection Administrators (PCA)**

• Full permissions across the entire organization
• Create/Delete projects
• Manage billing
• Manage agent pools
• Manage extensions
• Full admin on all projects and repos
• Override all permissions

**Usage**: Only a few people should be PCA (e.g., DevOps Lead).

## **1.2. Project Collection Build Administrators**

• Manage build resources across all projects
• Manage agent pools
• Configure pipelines
• Can control parallel jobs

## **1.3. Project Collection Service Accounts**

• Used by system processes (internal)
• Should never be modified

============================================================
**2. PROJECT-LEVEL GROUPS & PERMISSIONS**
============================================================

## **2.1. Project Administrators**

• Full control of the project
• Manage teams
• Manage repos
• Manage pipelines
• Manage boards
• Create/Edit/Delete service connections
• Edit project settings

**Usage**: DevOps Admin / Project Leads.

## **2.2. Contributors**

• Read/Write permissions to work items
• Edit code (push commits)
• Create branches
• Create pipelines (if enabled)
• Create PRs
• Update boards

**Usage**: Developers, Testers.

## **2.3. Readers**

• Read-only access to everything
• Can view code, pipelines, artifacts, boards

**Usage**: Management, Auditors.

## **2.4. Build Administrators (Project Level)**

• Manage build pipelines
• Edit YAML pipelines
• Manage variable groups
• Approve pipeline permissions (if enabled)

## **2.5. Release Administrators**

• Manage release pipelines
• Approve/Reject releases
• Edit CD pipelines

============================================================
**3. REPO-LEVEL PERMISSION GROUPS**
============================================================

(Applies to each repo inside a project)

## **3.1. Repo Administrators (Administer Git Repositories)**

• Delete repo
• Rename repo
• Force push
• Edit branch policies
• Configure security
• Create/Delete tags

## **3.2. Contributors**

• Push commits
• Create branches
• Create PRs
• Comment on code
• Resolve conflicts

## **3.3. Readers**

• View repo
• Clone repo
• Review PRs (no write)

============================================================
**4. PIPELINE-LEVEL PERMISSIONS**
=================================

## **4.1. Pipeline Administrators**

• Edit YAML
• Edit Classic pipeline
• Manage pipeline variables
• Manage triggers
• Grant access to resources
• Delete pipeline

## **4.2. Pipeline Contributors**

• Run pipeline
• Create pipelines
• Edit pipeline (if allowed)

## **4.3. Readers**

• View pipeline
• See logs

============================================================
**5. AGENT POOL PERMISSION GROUPS**
===================================

## **5.1. Administrator**

• Manage agent pool
• Add/remove agents
• Configure pool security

## **5.2. Service Account**

• The account used by build agents
• Should not be modified

## **5.3. Reader**

• View agents
• View pool usage

============================================================
**6. ENVIRONMENT PERMISSIONS (DEPLOYMENT)**
============================================================

## **6.1. Environment Administrators**

• Edit environment
• Configure resources (VMs, Kubernetes, etc.)
• Edit approvals
• Assign security

## **6.2. Users**

• Deploy to environment (if allowed)
• View resources

============================================================
**7. COMMON ENTERPRISE SCENARIOS**
==================================

## **Scenario 1: Standard DevOps Team**

• DevOps Lead → Project Administrator
• Developers → Contributors
• QA/Testers → Contributors
• Managers → Readers

## **Scenario 2: Strict Repo Control**

• Repo Admin → Only 1–2 people
• Developers → No force push, branch policy enforced

## **Scenario 3: Secure Production Deployment**

• Prod Environment Admin → DevOps Lead
• Deployment permission → Only Release Managers
• Approvers → Change Manager + Security Team

==================================================================================================================