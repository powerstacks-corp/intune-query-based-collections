# Intune Query-Based Collections

A Logic App and Azure Functions implementation of **query-based device collections for Microsoft Intune** — populating Entra ID security groups based on installed software, configuration state, and other inventory signals from Power BI / BI for Intune.

This is the ConfigMgr-style "dynamic collection" pattern brought to Intune: write a query (against your BI for Intune semantic model), get a fresh group membership in Entra ID on a configurable schedule, deploy applications and policies against the resulting group like any other Entra ID security group.

This repository is published by **PowerStacks** for **BI for Intune** customers.

> **Note:** Code is being uploaded by the maintainer. When complete the repository will contain the Logic App ARM/Bicep template, the Azure Function code, and a deployment guide.

**Companion blog post:** [How to create query-based collections in Intune](https://powerstacks.com/blog/how-to-create-query-based-collections-in-intune/)

---

## What's included (planned)

| Component | Purpose |
|-----------|---------|
| Logic App | Orchestrates the query-against-Power-BI-then-update-Entra-ID-group flow on a schedule |
| Azure Function (DAX query runner) | Executes a parameterized DAX query against your Power BI semantic model and returns the matching device or user IDs |
| Azure Function (Entra ID group reconciler) | Compares the query result to current group membership and reconciles via Microsoft Graph |
| ARM / Bicep templates | One-click deploy into a customer-managed Azure subscription |
| Sample DAX queries | Common patterns: "devices missing a specific app", "devices with risky config", "users in a country" |

## Prerequisites (planned)

- An active **BI for Intune** deployment (the queries run against its Power BI semantic model)
- An Azure subscription
- Permissions to create an app registration with `Group.ReadWrite.All` and `User.Read.All` on Microsoft Graph
- Permissions to create the Entra ID security group(s) that this solution will manage

---

## License

[MIT](LICENSE) - use, modify, and share freely.

## Maintainer

Maintained by **PowerStacks**. Issues and pull requests welcome on the [issue tracker](../../issues).
