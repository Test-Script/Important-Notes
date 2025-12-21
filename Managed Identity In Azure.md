======================================= How to create managed Identities ==============================

Purpose of managed identities:

In-order to avail the access of Azure Services, We require the managed Identies. Creating alone will not give us the access of services, Global Administrator has to assign the required to roles to identities.

There are types of Managed Identities:

User assigned (Recommend)
System Assigned (lifecycle linked with the resource)

Azure Portal --> Managed Identity --> Create One --> Upon creation assign the required Roles (Access Control IAM).

Roles : 

Job Function Roles

Priviledged Administrator Roles : Reader, Contributor, User Access Administrator.

Feature of Managed Identities:

Federated Credentials : Will be provide to us, In-order to avail the token based authentication.

1. Gives the Token based authentication to Github Repositories, In-order to make the changes into Azure Resource.

2. Gives the Token Based Authentication to Kubernetes Service Account to avail the capabilities to make the changes into Azure Resources.

3. Other Third-Party Apps can avail the token based authentication as well.

=========================================== Service Principles ========================================


