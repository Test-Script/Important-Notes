
============================ Azure Ecosystem Revising ================================

Azure Policy can be assigned at the tenant root, management group, subscription, or resource group level. Policies inherit downward, so resources automatically comply with higher-level governance. This enables centralized control while allowing flexibility at lower scopes.

[Azure WhiteBoard Shared Here](https://wbd.ms/share/v2/aHR0cHM6Ly93aGl0ZWJvYXJkLm1pY3Jvc29mdC5jb20vYXBpL3YxLjAvd2hpdGVib2FyZHMvcmVkZWVtLzY4MDNlMjE3MmUzODRlYzJiY2ZjZGNiNjE0MjYxODdhX0JCQTcxNzYyLTEyRTAtNDJFMS1CMzI0LTVCMTMxRjQyNEUzRF8xMjkyZDNkNy1kYjg3LTQ2NTYtODVkMy04ZTk1MTgyOWVlMmE=)

https://azure.microsoft.com/en-us/updates?id=542455 : Latest Update Information of Azure.

Tenant (Entra ID)
 └─ Management Group
     └─ Subscription
         └─ Resource Group
             └─ Resources

======================== Azure Policy supports assignment at all scopes ======================


| Scope                   | Policy Applicable? | Notes                                             |
| ----------------------- | ------------------ | ------------------------------------------------- |
| **Tenant Root**         | ✅ Yes              | Used for org-wide governance                      |
| **Management Group**    | ✅ Yes              | Best practice for enterprise guardrails           |
| **Subscription**        | ✅ Yes              | Subscription-level compliance                     |
| **Resource Group**      | ✅ Yes              | Workload-specific controls                        |
| **Individual Resource** | ❌ No               | Policy evaluates resources, not assigned directly |

============================== How Inheritance Works ==========================================

- Policies are inherited downward

- Lower scopes cannot override stricter policies from higher scopes

============================== Enterprise Best Practices =============================

| Level                | Typical Policy Usage                               |
| -------------------- | -------------------------------------------------- |
| **Tenant / Root MG** | Security baseline (no public IPs, allowed regions) |
| **Management Group** | Compliance (ISO, SOC, naming, tagging)             |
| **Subscription**     | Cost controls, SKU limits                          |
| **Resource Group**   | Workload-specific rules                            |


================================= Primary Effects of Policy =========================

Primary types of Azure Policy effects:

    Deny: Prevents resource creation or modification if it violates the policy, enforcing strict compliance.

    Audit: Creates a compliance record in the Activity Log for non-compliant resources, but allows the resource to be created/updated.

    Modify: Adds, updates, or removes tags/properties on matching resources during creation or update, useful for enforcing standards.

    DeployIfNotExists (DINE): Deploys a template (like a diagnostic setting) if a related resource isn't found or compliant.

    Append: Adds specified tags or properties to a resource during creation or update.

    AuditIfNotExists (AINE): Audits for the existence of related resources (e.g., extensions, security settings) that should be present.

    DenyAction: Prevents specific Resource Manager operations (like delete) on resources matching the policy.

    Manual: Allows for self-attestation of compliance, useful for regulatory standards where manual sign-off is needed.

    Disabled: Turns off evaluation for a policy assignment; resources aren't evaluated or marked as compliant/non-compliant.

    AddToNetworkGroup: Adds a network interface to a specified Network Security Group (NSG) group.
    Mutate: (Preview) A more advanced effect for complex resource data manipulation. 