=====================================================================
AZURE DEVOPS AGENT – COMPLETE INTERNALS (SELF-HOSTED)
=====================================================================

1. WHAT IS AN AZURE DEVOPS AGENT

An Azure DevOps Agent is a worker that executes pipeline jobs defined in Azure DevOps.

Two types:
• Microsoft-hosted agent (managed by Microsoft)
• Self-hosted agent (installed and managed by you)

This document focuses on **Self-Hosted Agents**.

2. AGENT INSTALLATION LOCATION (ROOT)

When you install a self-hosted agent, you choose a root directory.

Example:
C:\azagent
/opt/azagent/
/home/azureagent/

This root directory is the **agent home**.

3. CORE AGENT DIRECTORY STRUCTURE

Inside the agent root directory:

.agent
.config
.credentials
.service
bin/
externals/
logs/
work/
_diag/
env.sh / run.sh (Linux)
run.cmd (Windows)

4. PURPOSE OF EACH DIRECTORY / FILE

## .agent

• Stores agent registration metadata
• Links agent to:
– Organization
– Agent Pool
– Agent Name

Do NOT delete unless reconfiguring agent.

## .config

• Agent configuration details
• Server URL
• Pool name
• Agent capabilities

Deleting this forces agent reconfiguration.

## .credentials

• Encrypted auth material
• PAT / OAuth token
• Service Principal info (if used)

Highly sensitive.
Never commit or copy.

## .service

• Exists if agent runs as a service
• Used by Windows Service / systemd (Linux)

## bin/

• Agent binaries
• Worker executables
• Pipeline engine

Updated automatically during agent upgrade.

## externals/

• Bundled dependencies
• Node.js
• Git
• PowerShell Core (sometimes)

Agent downloads tools here.

## logs/

• Agent runtime logs
• Connectivity issues
• Registration errors

Useful for troubleshooting:
Agent.Listener.log
Agent.Worker.log

## _diag/

• Diagnostic logs
• Crash dumps
• Low-level telemetry

Enable verbose troubleshooting here.

## env.sh / run.sh / run.cmd

• Agent startup scripts
• Used to run interactively

5. WORK DIRECTORY (MOST IMPORTANT)

Path:
<agent_root>/work/

Example:
C:\azagent\work
/opt/azagent/work

This is where **all pipeline execution happens**.

6. WORK DIRECTORY STRUCTURE

Inside work/:

1/
2/
3/
...
_s/
_t/
_a/

Each **numbered folder = one pipeline workspace**.

7. NUMBERED PIPELINE DIRECTORIES (1/, 2/, 3/)

Example:
work/1/
work/2/

Each number represents:
• A pipeline job allocation
• Reused between runs (unless cleaned)

Inside work/1/:

a/
b/
s/
TestResults/
.tmp/

8. PIPELINE SUBDIRECTORIES EXPLAINED

## s/  (SOURCE DIRECTORY)

• Source code checkout location
• Maps to:
$(Build.SourcesDirectory)

Example:
work/1/s/

If multiple repos:
work/1/s/repo1
work/1/s/repo2

## a/  (ARTIFACT STAGING)

• Staging area for build artifacts
• Maps to:
$(Build.ArtifactStagingDirectory)

Artifacts copied here before publishing.

## b/  (BINARIES)

• Compiled outputs (legacy usage)
• Maps to:
$(Build.BinariesDirectory)

Often unused in modern pipelines.

## TestResults/

• Test execution results
• TRX, JUnit, coverage reports

## .tmp/

• Temporary files
• Task-level scratch data

9. GLOBAL WORK FOLDERS

## _s/

• Reserved for internal agent usage
• Do not modify

## _t/

• Tool cache
• Installed tools:
– Terraform
– Helm
– Node
– Kubectl

Maps to:
$(Agent.ToolsDirectory)

## _a/

• Global artifact download directory
• Used by:
DownloadPipelineArtifact task

Maps to:
$(Pipeline.Workspace)

10. IMPORTANT PIPELINE VARIABLES → DIRECTORY MAP

$(Agent.WorkFolder)              → work/
$(Pipeline.Workspace)            → work/_a
$(Build.SourcesDirectory)        → work/1/s
$(Build.ArtifactStagingDirectory)-> work/1/a
$(Build.BinariesDirectory)       → work/1/b
$(Agent.ToolsDirectory)          → work/_t
$(System.DefaultWorkingDirectory)-> work/1/s

11. PIPELINE EXECUTION FLOW (INTERNAL)

1. Agent receives job from Azure DevOps
2. Workspace (work/1) allocated
3. Repo cloned into s/
4. Tasks executed in sequence
5. Artifacts staged in a/
6. Results uploaded
7. Workspace reused or cleaned

12. WORKSPACE CLEANUP BEHAVIOR


By default:
• Workspace is reused
• Faster builds
• Risk of stale files

Enable clean checkout:
checkout: self
clean: true

Or pipeline setting:
Workspace → Clean: all

13. SELF-HOSTED AGENT VS MICROSOFT-HOSTED

Self-Hosted:
• Persistent workspace
• Full OS control
• Faster for large repos
• Requires maintenance

Microsoft-Hosted:
• Ephemeral VM
• Fresh environment every run
• No persistence


14. SECURITY CONSIDERATIONS


• Restrict agent machine access
• Never run agent as root unless required
• Separate prod & non-prod agents
• Use least-privileged Service Principals
• Rotate PATs regularly


15. AGENT CAPABILITIES


Capabilities determine job eligibility:
• OS
• Installed tools
• Custom user capabilities

View in:
Agent Pool → Agent → Capabilities

16. MULTIPLE AGENTS ON SAME MACHINE

Best practice:
• Separate agent root directories
• Separate service accounts
• Avoid shared work folders

Example:
C:\azagent01
C:\azagent02\

17. COMMON ISSUES & ROOT CAUSE

• Disk full → work folder not cleaned
• Permission denied → agent user mismatch
• Stale terraform plan → reused workspace
• Tool version conflict → cached in _t/

18. BEST PRACTICES (PRODUCTION)

• Dedicated VM per environment
• One agent pool per environment
• Enable workspace cleanup for infra pipelines
• Monitor disk usage
• Patch agent regularly
• Log rotation enabled

19. WHEN TO DELETE AGENT FOLDERS

Safe to delete:
• work/*
• logs/*

Do NOT delete:
• .credentials
• .config
• .agent

20. QUICK MENTAL MODEL

Agent Root
├── Agent Engine
├── Auth & Config
├── Logs
└── Work
├── Source Code
├── Artifacts
├── Tools
└── Temp Data

=====================================================================