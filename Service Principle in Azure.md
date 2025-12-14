

================================= How to Create Service Principal in Azure ===================================

--------------------------------------- Method 1: Azure Portal -----------------------------------------------

1. Login to Azure Portal
2. Search: App Registrations
3. Click: New Registration
4. Enter the following:
   Name: my-sp
   Supported Account Types: Single Tenant
5. Click: Register
6. Open: Certificates & Secrets
7. Create: New Client Secret
8. Copy and save:
   Application (Client) ID
   Directory (Tenant) ID
   Client Secret Value
9. Assign Role from:
   Subscriptions > IAM > Add Role Assignment

--------------------------------------- Method 2: Azure CLI ---------------------------------------
Login command:
az login

Create Service Principal:
az ad sp create-for-rbac --name "myServicePrincipal" --role Contributor --scopes /subscriptions/<SUBSCRIPTION_ID>

Example Output:
appId = Client ID
password = Client Secret
tenant = Tenant ID

Create SP with resource group scope:
az ad sp create-for-rbac --name "mySPcustom" --role Reader --scopes /subscriptions/<SUB_ID>/resourceGroups/<RG_NAME>

Reset Client Secret:
az ad sp credential reset --name "<Client_ID>"

Assign Role later:
az role assignment create --assignee <Client_ID> --role Contributor --scope /subscriptions/<SUB_ID>

--------------------------------------- Method 3: PowerShell ---------------------------------------
Login:
Connect-AzAccount

Create Service Principal:
$sp = New-AzADServicePrincipal -DisplayName "myServicePrincipal"

Assign Role:
New-AzRoleAssignment -RoleDefinitionName "Contributor" -ServicePrincipalName $sp.ApplicationId -Scope "/subscriptions/<SUB_ID>"

Retrieve Info:
Client ID: $sp.ApplicationId
Object ID: $sp.Id

Important Notes:
Store Client Secret securely (Key Vault recommended)
Do not share credentials publicly
Rotate secrets periodically

======================================================================================================================