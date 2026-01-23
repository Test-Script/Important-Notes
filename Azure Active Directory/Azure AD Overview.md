======================= Azure AD Overview ====================

1. What is Azure AD and how it related to AD.
2. Authenctication options to Azure AD Cloud auth, PTA and federation.
3. Seamless and Single Sign On.
4. Fedaration with enteripse applicattion. 
5. Conditinal Access and MFA
6. PIM.
7. How Azure AD relates to Azure Subscription.

https://youtu.be/EUVKEhiHYG0?si=Ofa_S1k180xYsBdT

Principle Domain you will get as soon as we created the Azure Account.

onmicrosoft.com  --> Primary Domain

Default & Immutable: yourcompany.onmicrosoft.com is created with your tenant and cannot be deleted.

Custom Domains: You can add your own domain (e.g., @yourcompany.com) and designate it as primary for user logins.

In essence, onmicrosoft.com is your tenant's foundational, built-in domain, but your actual business domain becomes the main one for daily user operations.


=================== Authentication Workflow ==================

Cloud : SAAS Application <---> Microsoft Entra ID (OIDC Oauth, SAML, etc)

Microsoft Entra ID <----> Office License.

Microsoft Entra ID <----> Azure Cloud Subscription.

Client ----> Federation (PASSWORD) <---> Azure AD <---> Service.

Conditional Access 