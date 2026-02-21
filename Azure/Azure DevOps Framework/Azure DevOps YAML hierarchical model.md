# Azure DevOps YAML Hierarchical Model

## YAML Schema Hierarchy (Authoring Perspective)

Azure DevOps YAML follows a strict hierarchical model:

Pipeline
 └── stages (collection)
       └── stage (single logical phase)
             └── jobs (collection)
                   └── job (execution unit)
                         └── steps (collection)
                               └── step (atomic action)
                    
# Always keep in mind the collection, execution unit, single logical phase.

====================================================================================================
                                Execution Granularity Comparison
====================================================================================================

| Layer | Scheduling Unit | Parallel? | Agent Isolation  | Dependency Support |
| ----- | --------------- | --------- | ---------------- | ------------------ |
| Stage | Yes             | Yes       | Logical          | Yes                |
| Job   | Yes             | Yes       | Physical agent   | Yes                |
| Step  | No              | No        | Shared workspace | Limited            |

====================================================================================================
                                    Mental Model for Brain Training
====================================================================================================

Question : What is the difference between stage, job and step?

- Stage → Logical lifecycle boundary
- Job → Smallest schedulable agent execution unit
- Step → Atomic command executed sequentially inside a job

====================================================================================================
                            Critical Concept: Isolation Boundaries
====================================================================================================

Stage Boundary

- Logical separation
- Can require approval
- Can define different environments
- Ideal for promotion (Dev → QA → Prod)

====================================================================================================
                                            Job Boundary
====================================================================================================

- New agent allocation
- Clean workspace
- Separate logs
- Parallel execution unit

====================================================================================================
                                            Step Boundary
====================================================================================================

- Same agent
- Same working directory
- Shared environment variables

====================================================================================================
                                        DevOps Design Pattern
====================================================================================================

Stage 1: CI
  Job 1: Build
  Job 2: Unit Test
  Job 3: Security Scan

Stage 2: CD-Dev
  Job: Deploy to Dev

Stage 3: CD-Prod
  Job: Deploy to Prod

====================================================================================================
                                    Advanced Comparison Table
====================================================================================================

| Feature             | Stage                   | Job                  | Step        |
| ------------------- | ----------------------- | -------------------- | ----------- |
| YAML Keyword        | stage                   | job                  | script/task |
| Execution Unit      | Logical phase           | Agent-bound unit     | Command     |
| Parallelism         | Yes                     | Yes                  | No          |
| Dependency Control  | Yes                     | Yes                  | Limited     |
| Manual Approval     | Yes                     | No                   | No          |
| Environment Binding | Yes                     | Yes (deployment job) | No          |
| Artifact Sharing    | Between stages          | Within stage         | Within job  |
| Failure Impact      | Fails downstream stages | Fails dependent jobs | Fails job   |

====================================================================================================
                                    Common Misconceptions
====================================================================================================

❌ “Stage and Job are same”

No.

Stage = Lifecycle boundary
Job = Compute execution unit

❌ “Steps can run in parallel”

No.

Parallelism only exists at job or stage level.

❌ “Multiple jobs share same workspace”

No.

Each job gets fresh agent workspace.

====================================================================================================
                            Architectural Insight (Important for Me Only)
====================================================================================================

Optimizing agent pools:

👉 Parallel jobs = parallel agent allocation
👉 Too many jobs = pool exhaustion
👉 Stage design impacts concurrency licensing

Pipeline throughput depends more on job design than stage design.

====================================================================================================
                                Differenatial Manner Of Learning
====================================================================================================

1. stages vs stage:

This is a top-level collection keyword.

    stages:
    - stage: Build
    - stage: Deploy

It is simply a list container, and It does not execute anything.

2. stage:

A stage is a logical boundary in pipeline execution.

    - stage: Build
      jobs:
      - job: Compile

A stage represents:

- A lifecycle phase (Build/Test/Deploy).
- A promotion boundary.
- A compliance boundary.
- A manual approval checkpoint.
- A rollback isolation layer.

Stage Execution Model:

| Property          | Behavior                         |
| ----------------- | -------------------------------- |
| Execution         | Sequential by default            |
| Parallel          | Possible if `dependsOn` modified |
| Isolation         | Has its own dependency graph     |
| Approval          | Can have environment approvals   |
| Artifact boundary | Common artifact exchange point   |

# Stage as DAG Node:

Internally Azure DevOps builds a Directed Acyclic Graph (DAG).

stages:
- stage: Build
- stage: Test
  dependsOn: Build
- stage: Deploy
  dependsOn: Test

Execution graph: Build → Test → Deploy

# Important To Note Here, If we remove the dependancy, All can run in parallel, and to make this happen by dependsOn Keyword.

Build
Test
Deploy

3. jobs vs job:

jobs:

# Importan to keep in mind : Container for multiple jobs.

jobs:
- job: LinuxBuild
- job: WindowsBuild

job:

# A job is the smallest schedulable unit assigned to an agent.

- job: BuildJob
  pool:
    vmImage: ubuntu-latest

## Key Characteristics of Job

| Property    | Behavior                                       |
| ----------- | ---------------------------------------------- |
| Runs on     | One agent                                      |
| Parallelism | Jobs inside same stage run parallel by default |
| Isolation   | Each job has clean workspace                   |
| Timeout     | Per-job configurable                           |
| Retry       | Job-level retry possible                       |

## Job Types

- Agent Job → Runs on build agent.
- Deployment Job → Used for environments.
- Server Job → Agentless (manual validation, REST calls).

## Job Dependency Graph

Inside a stage:

jobs:
- job: A
- job: B
  dependsOn: A

Creates:

    A → B

If no dependency:

A
B   (parallel)

4. steps vs step:

steps:

    Container for atomic execution instructions.

steps:
- script: echo Hello
- task: UsePythonVersion@0

step: 

A step is: The smallest executable action inside a job.

Types of steps:

- Script (bash/powershell)
- Task
- Checkout
- Template inclusion
- Download artifact

## Step Execution Model

Inside a job: Step1 → Step2 → Step3 

## Steps are always sequential.

There is no parallel execution inside a single job.