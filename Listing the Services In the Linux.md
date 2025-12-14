On Linux (Command Line)
All Services (Active & Inactive): sudo systemctl list-units --type=service --all.
Only Enabled (Starts on Boot): systemctl list-unit-files --type=service --state=enabled.
Running Services: systemctl list-units --type=service --state=running
Older Systems (SysVinit): service --status-all. 


vsts.agent.tejastarghale47.Self_Agent_Pool.localhost.service

jobs:
- job: myJob
  workspace:
    clean: outputs | resources | all # Choose what to clean
  steps:
  # ... pipeline steps