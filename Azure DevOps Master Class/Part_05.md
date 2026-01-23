================================= DevOps Master Class - Part 4 - CI/CD ===================================

# PLAN --> DEVELOP --> COLLABORATION --> DELIVERY --> OPERATE

# Build --> Test --> Artifacts --> Upload (Azure Container Registry)

## Continuous Integration
# Security Check before deployment packages
# - Check Codes
# - Secrets
# - Dependencies Scanning
# - Remediate the Dependency Vulnarabilities
# Docker Container Security at the Registry

# In Between CI, and CD, There should be always ready Infrastructure Provisioned by the Terraform, Biceps (Declarative, Version Control).

# All Infrastructure should be production consistent.

# If Infrastructure is VM based than with the ansible will have to configure it's old dependencies.

## Continuous Delivery & Deployment

# New Artifacts Release (Should be refered by the all environments i.e. Test, Staging, and Production)

# Each Stage There should be one Infrastructure as Code Facility to Maintain consistent environment.

# Environments --> Test (Test Gate)--> Staging (Release Gate)--> Producation.

# Environment Specific Gates We should have at each stage for Test, QA, Staging, and Production.

# There are two Kinds of the Testing : Code Test, and Infrastructure testing.

======================= Environment Testing  ====================

At each there should be an testing executed for the environment.

- Funcationality Test
- Security
- Compliant
- Performanace
  - Variance Percentagewise.
- Smoke Test at Production.

# Production Rollout 

# |1|2|3|4|  --> Segament of the Producation release for the new code-release.

# In-Place Upgrade

    - Deployment Slots : Simple Required Downtime
    - Progressive (Rings) : Control , Complex Take time.

    - CANARY (%)    : Control Simplicity , Time / Complexity

    - BLUE/GREEN : Simpler, Resource USE, Most big deal for Cloud.

# Azure DevOps Pipeline

GUI Based Pipeline Structure

- BUILD  : CI

- RELEASE : CD

YAML Pipelines

- NAME
- TRIGGER
- POOL
- [STAGES]
    - [DependsOn]
- JOBS
- pool
- [POOL]
    - [ENVIRONMENT]  ---> Security, Resources, Approvals/Checks, Track.
    - STEPS
    - TASKS
Multi-Stage Pipeline : CI + CD


Azure Pipeline YAML STRUCTURE
=============================

Pipeline
│
├── trigger
│    └── Defines when the pipeline runs (branch / path / none)
│
├── variables
│    └── Global variables available across all stages
│
├── stages
│    │
│    ├── stage: <StageName>
│    │    │
│    │    ├── displayName: "<Readable Stage Name>"
│    │    │
│    │    ├── dependsOn: <PreviousStage>
│    │    │
│    │    ├── condition: <Stage Condition>
│    │    │
│    │    └── jobs
│    │         │
│    │         ├── job: <JobName>
│    │         │    │
│    │         │    ├── displayName: "<Readable Job Name>"
│    │         │    │
│    │         │    ├── pool
│    │         │    │    └── vmImage: 'ubuntu-latest'
│    │         │    │
│    │         │    ├── variables
│    │         │    │    └── Job-level variables
│    │         │    │
│    │         │    └── steps
│    │         │         │
│    │         │         ├── script: |
│    │         │         │     echo "Shell Script Execution"
│    │         │         │
│    │         │         ├── bash: |
│    │         │         │     echo "Bash Script"
│    │         │         │
│    │         │         ├── powershell: |
│    │         │         │     Write-Host "PowerShell Script"
│    │         │         │
│    │         │         ├── task: AzureCLI@2
│    │         │         │     inputs:
│    │         │         │         scriptType: bash
│    │         │         │         scriptLocation: inlineScript
│    │         │         │         inlineScript: |
│    │         │         │             az --version
│    │         │         │
│    │         │         └── checkout: self
│    │         │
│    │         └── job: <AnotherJob>
│    │
│    └── stage: <AnotherStage>
│
└── resources
     └── External repositories / pipelines


============================= Key Hierarchy Rules (Important) ========================

stages
 └── jobs
      └── steps
           └── scripts / tasks

============================ Minimal Working YAML Example ==========================

trigger:
- main

stages:
- stage: Build
  jobs:
  - job: BuildJob
    pool:
      vmImage: ubuntu-latest
    steps:
    - script: echo "Building the application"

- stage: Deploy
  dependsOn: Build
  jobs:
  - job: DeployJob
    steps:
    - script: echo "Deploying the application"


===================================== Important Notes =======================================

• A pipeline can have multiple stages
• A stage can have multiple jobs
• A job can run on only ONE agent
• Steps execute sequentially inside a job
• Stages can run in parallel unless restricted using dependsOn
• Scripts are always executed inside steps

==================================== When to Use What ======================================

Use STAGES  → Environment separation (Build / Test / Deploy)
Use JOBS    → Parallel execution or different agents
Use STEPS   → Sequential execution logic
Use SCRIPTS → Actual command execution

=================== Difference Between Continous Deployment, and the Continous Delivery =====================================

Continous Delivery → It ensures that we have production ready code for the deployment.

Continous Deployment is completely depends on the Continous Delivery as It is ensuring for deployment we are having the ready code.

Continous Deployment Pushes the Code to the Production environment.