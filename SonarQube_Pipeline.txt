trigger:
  - main

pr:
  - main

pool:
  name: Self_Agent_Pool

variables:
  PYTHON_VERSION: '3.12.3'
  SONAR_PROJECT_KEY: 'SonarQube_SonarQube_Test_348483c9-0cbe-4350-933e-723b2231a412'
  SONAR_PROJECT_NAME: 'SonarQube_Test'
  COVERAGE_FILE: 'coverage.xml'

steps:

# ----------------------------
# Checkout Code
# ----------------------------
- checkout: self
  clean: true

# # ----------------------------
# # Python Setup
# # ----------------------------
# - task: UsePythonVersion@0
#   inputs:
#     versionSpec: '$(PYTHON_VERSION)'
#     addToPath: true

# ----------------------------
# Dependencies
# ----------------------------
- script: |
    python3 -m venv venv
    source venv/bin/activate

    python -m pip install --upgrade pip
    pip install -r requirements.txt
    pip install pytest pytest-cov
  displayName: Install Dependencies (venv)


# ----------------------------
# Tests + Coverage
# ----------------------------
- script: |
    source venv/bin/activate
    python -c "import django; print('Django loaded from:', django.__file__)"
    pytest --cov=. --cov-report=xml:coverage.xml
  displayName: Run Tests with Coverage

# ----------------------------
# SonarQube Prepare
# ----------------------------
- task: SonarQubePrepare@7
  displayName: Prepare SonarQube Analysis
  inputs:
    SonarQube: 'Sonar_Qube_Azure'
    scannerMode: 'CLI'
    configMode: 'manual'
    cliProjectKey: '$(SONAR_PROJECT_KEY)'
    cliProjectName: '$(SONAR_PROJECT_NAME)'
    cliSources: '.'
    extraProperties: |
      sonar.python.coverage.reportPaths=$(COVERAGE_FILE)
      sonar.sourceEncoding=UTF-8
      sonar.branch.name=$(Build.SourceBranchName)

# ----------------------------
# SonarQube Analyze
# ----------------------------
- task: SonarQubeAnalyze@7
  displayName: Run SonarQube Analysis

# ----------------------------
# SonarQube Publish
# ----------------------------
- task: SonarQubePublish@7
  displayName: Publish Quality Gate Result
  inputs:
    pollingTimeoutSec: '300'
