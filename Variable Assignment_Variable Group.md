Referance Link for the Azure DevOps

https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=azure-pipelines-ui%2Cyaml


###############################################
# Using Library Variables in Azure DevOps YAML
###############################################

# Your pipeline (corrected & annotated)
# -------------------------------------

trigger:
- main

pool:
  name: Self_Agent_Pool

# 1) Link the Variable Group from Library
variables:
- group: My-Variable-Group     # must match Library group name EXACTLY

stages:
- stage: SystemCheck
  displayName: "System & Docker Health Check"
  jobs:

  # =========================
  # Job 1: Host & Network
  # =========================
  - job: HostAndPing
    displayName: "Host & Network Information"
    steps:

    # A) Use Library Variable in script
    - script: |
        echo "User: $(username)"
      displayName: "Print Variable from Library"

    # B) Use same variable in env section (good for secrets)
    - script: |
        echo "Env user: $MY_USER"
      env:
        MY_USER: $(username)
      displayName: "Use Library Variable as ENV"

    # C) Use in a task input example (pseudo example)
    # - task: Bash@3
    #   inputs:
    #     targetType: 'inline'
    #     script: |
    #       echo "Username is: $(username)"

###############################################
# IMPORTANT POINTS
###############################################

# 1️⃣ Variable Group Name
# - In Library → Variable groups → "My-Variable-Group"
# - The name must be exactly same as in YAML:
#       variables:
#       - group: My-Variable-Group

# 2️⃣ Variable Name
# - In the variable group, create a variable, e.g.:
#       Name  : username
#       Value : tejas_admin
# - Use it in pipeline as:
#       $(username)

# 3️⃣ Case Sensitivity
# - Azure DevOps variable names are usually case-insensitive
#   BUT keep same style to avoid confusion:
#   If Library variable is "UserName", prefer:
#       $(UserName)

# 4️⃣ Using in displayName, script, condition
# - displayName: "Running for $(username)"
# - script: echo $(username)
# - condition: and(succeeded(), eq(variables['username'], 'tejas_admin'))

###############################################
# COMPILE-TIME vs RUNTIME SYNTAX (ADVANCED)
###############################################

# Usually this is enough:
#     $(username)
#
# But for template or conditional logic you may see:
#     ${{ variables.username }}
#
# Example:
# - ${{ if eq(variables['username'], 'tejas_admin') }}:
#   - script: echo "Admin user"

###############################################
# QUICK CHECKLIST IF VALUE IS EMPTY
###############################################

# If echo $(username) prints nothing:
# 1) Check Library → Variable groups:
#    - Group name is "My-Variable-Group"
#    - Variable "username" exists
# 2) Check "Allow access to all pipelines" is enabled
#    OR variable group is explicitly linked in the pipeline UI.
# 3) Re-run pipeline after changes.

###############################################
# TL;DR FOR YOUR PIPELINE
###############################################

# In any step inside that YAML you can just do:
#   echo $(username)
# or
#   env:
#     MY_USER: $(username)
# or
#   displayName: "Task for $(username)"
###############################################
