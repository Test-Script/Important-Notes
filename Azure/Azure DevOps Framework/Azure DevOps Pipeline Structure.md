==========================================================================================================
                                    Azure DevOps Pipeline Structure
==========================================================================================================

Azure DevOps Pipelines are defined using YAML (recommended)

Pipeline
 ├── Trigger
 ├── Variables
 ├── Stages
 │     ├── Jobs
 │     │     ├── Steps
 │     │     │     ├── Tasks / Scripts
 │     │     │     └── Templates
 ├── Environments
 └── Artifacts

==========================================================================================================
                                            Agent Pool
==========================================================================================================

1. Microsoft-hosted agents

2. Self-hosted agents

3. Scale set agents

==========================================================================================================
                                        Security Architecture
==========================================================================================================

1. Service connections

2. Managed Identity

3. RBAC

4. Variable secrets

5. Approval checks

6. Environment protections

==========================================================================================================
                                Real-World DevOps Pipeline Lifecycle
==========================================================================================================

- Code Push →
- CI Trigger →
- Build →
- Unit Test →
- Code Quality →
- Security Scan →
- Artifact Publish →
- Deploy Dev →
- Integration Test →
- Approval →
- Deploy Prod →
- Monitoring Hook

==========================================================================================================
                                    Structural Best Practices (Enterprise)
==========================================================================================================

✔ Separate CI & CD logically
✔ Use multi-stage YAML
✔ Use templates
✔ Implement environment approvals
✔ Use Key Vault for secrets
✔ Enable branch protection
✔ Implement artifact versioning
✔ Use caching to optimize builds
✔ Use parallel jobs wisely

==========================================================================================================
                                        Core Components
==========================================================================================================

1. Trigger Section

trigger:
  branches:
    include:
      - main
      - develop

pr:
  branches:
    include:
      - main

Types:
- CI Trigger
- PR Trigger
- Scheduled Trigger
- Manual Trigger

2. Variables

Used for parameterization.

variables:
  buildConfiguration: 'Release'
  vmImage: 'ubuntu-latest'

Types:
- Pipeline variables
- Variable groups
- Key Vault secrets
- Runtime parameters

3. Stages (Logical Boundary)

Represents lifecycle phase.

- Build
- Test
- Security Scan
- Deploy Dev
- Deploy Prod

stages:
- stage: Build
  jobs:
  - job: BuildJob

✔ Stages can have:
- Approvals
- Gates
- Conditions
- Dependencies

4. Jobs (Execution Unit):

Runs on an agent.

jobs:
- job: BuildJob
  pool:
    vmImage: ubuntu-latest

Job Types:
- Agent job
- Deployment job
- Server job (agentless)

# Important Note : Always Keep in Mind Jobs inside same stage run parallel by default.

5. Steps (Inside Job):

# Sequential execution.

steps:
- script: echo "Building application"
- task: UsePythonVersion@0

Steps can be:
- Script (bash/powershell)
- Predefined task
- Template
- Checkout

6. Tasks

# Pre-built reusable components.

Examples:

- AzureCLI@2
- Docker@2
- PublishBuildArtifacts@1
- KubernetesManifest@1

7. Artifacts

# Used to pass output between stages.

- task: PublishBuildArtifacts@1

Than later:

- download: current

8. Environments (For Deployment Control)

# Important Note Keep in Mind While using the inside in deployment jobs.

- deployment: DeployWeb
  environment: 'Prod'

Provides:

- Approval gates
- Resource tracking
- Deployment history
- Security boundary

==========================================================================================================
                                    Enterprise-Grade Multi-Stage Pipeline
==========================================================================================================

trigger:
- main

variables:
  vmImage: ubuntu-latest

stages:

- stage: Build
  jobs:
  - job: BuildApp
    pool:
      vmImage: $(vmImage)
    steps:
    - checkout: self
    - script: echo "Building App"
    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: drop

- stage: Deploy_Dev
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployDev
    environment: Dev
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
          - script: echo "Deploying to Dev"

- stage: Deploy_Prod
  dependsOn: Deploy_Dev
  condition: succeeded()
  jobs:
  - deployment: DeployProd
    environment: Prod
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
          - script: echo "Deploying to Prod"