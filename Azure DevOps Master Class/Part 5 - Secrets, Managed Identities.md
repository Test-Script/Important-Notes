===============================  DevOps Master Class - Part 5 - Secrets ====================================

![Azure Authentication Secret Storage](image.png)

NEVER PUT YOUR SECRET IN THE CODE

Secret Rotation Policy : Every Quarter

Azure Key Vault : Centralize Secret store for storing all the secrete resources.

Azure Active Directory Authentication Methods

![Authentication Workflow](image-1.png)


Scenario :  

1. Virtual Machine : 10
2. Storage Account : 1

Instead of giving the RBAC access of Storage Account to each Virtual machine, We will give it by using the User Assigned Managed Identity.

With User Assigned Managed Identity, We can give access to n number of resources.

Azure Key Vault : 

1. Secret (R/W)
2. Keys
3. Certificates

- Versioning 
- Centralized
- RBAC

====== Project / Organizational Level Secret.

Never Put your secrets in Code.

Like Never Write Your Secret In The Disk of Host Where your code is writing.

- Secret
    - 