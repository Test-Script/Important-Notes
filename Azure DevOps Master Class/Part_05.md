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

# If Infrastructure is VM based than with the ansible will have to configure it's dependencies.

## Continuous Delivery & Deployment

# New Artifacts Release (Should be refered by the all environments i.e. Test, Staging, and Production)

# Environments --> Test (Test Gate)--> Staging (Release Gate)--> Producation.



