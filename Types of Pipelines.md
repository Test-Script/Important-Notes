Different Types of Pipelines in Azure DevOps
============================================

Azure DevOps offers 2 Major Types of Pipelines:

1️⃣ Build Pipeline (Classic: Build / YAML: CI Pipeline)
------------------------------------------------------
Purpose: Compile code, run builds, run unit tests, package artifacts
Triggers: Code commits, Pull Requests, Scheduled triggers
Outputs: Build Artifacts (e.g. .zip, NuGet packages, docker images)
Tools: MSBuild, Maven, Gradle, .NET, Node.js, Docker

Example Tasks:
- Restore dependencies
- Compile application
- Run unit tests
- Code quality checks (SonarQube)
- Generate build artifacts for release

--------------------------------------------

2️⃣ Release Pipeline (Classic Release / YAML: CD Pipeline)
----------------------------------------------------------
Purpose: Deploy build artifacts to environments (Dev/Test/Prod)
Triggers: After a successful Build OR manual approval
Supports: Approvals, Gates, Environment strategies
Outputs: Deployment of application or infrastructure

Example Tasks:
- Deploy WebApp to Azure/AWS
- Database migration script execution
- Infra deploy using ARM/Bicep/Terraform
- Configuration changes

============================================

Pipeline Categories by Deployment Strategy
------------------------------------------
a) CI (Continuous Integration)
   - Every commit triggers build + test
   - Fast feedback to developers

b) CD (Continuous Delivery)
   - Automated deploy to staging / pre-prod
   - Production deploy usually requires approval

c) Continuous Deployment
   - Fully automated deployment to Production
   - No manual approvals

============================================

Pipeline Types Based on Technology
----------------------------------
✔ Build Pipelines for:
   - .NET / Java / Python / Node.js
   - Docker image build pipelines

✔ Deployment Pipelines for:
   - Azure Web Apps / App Services
   - Kubernetes (AKS/EKS)
   - Virtual Machines
   - SQL Database deployments, etc.

✔ Infrastructure Pipelines:
   - Terraform Pipelines
   - ARM/Bicep Deployment Pipelines
   - Ansible / PowerShell pipelines

============================================

Pipeline Formats in Azure DevOps
--------------------------------
1️⃣ YAML Pipelines (Modern)
   - Pipeline as Code
   - Version controlled in repo
   - CI & CD in single YAML file
   - Recommended for DevOps automation

2️⃣ Classic Pipelines (GUI)
   - Drag-and-drop editor
   - Build and Release separate
   - Used for older projects

============================================

Summary Table
--------------------------------------------
| Pipeline Type      | Used For                         | Format          |
|-------------------|----------------------------------|-----------------|
| Build Pipeline     | Compile + Test + Create Artifacts| YAML/Classic    |
| Release Pipeline   | Deploy Artifacts to environments | Classic only    |
| CI Pipeline        | Auto Build on code changes       | YAML/Classic    |
| CD Pipeline        | Auto Deploy to environments      | YAML/Classic    |
| Infra Pipeline     | Provision Cloud Infrastructure   | YAML/Classic    |

============================================
End of Notes
