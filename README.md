# Intune Query-Based Collections

A Logic App and Power Automate flow that implement **query-based device collections for Microsoft Intune** — populating Entra ID security groups based on installed software, configuration state, and other inventory signals from Power BI / BI for Intune.

This is the ConfigMgr-style "dynamic collection" pattern brought to Intune: write a query (against your BI for Intune semantic model), get a fresh group membership in Entra ID on a configurable schedule, then deploy applications and policies against the resulting group like any other Entra ID security group.

This repository is published by **PowerStacks** for **BI for Intune** customers.

**Companion blog post:** [How to create query-based collections in Intune](https://powerstacks.com/blog/how-to-create-query-based-collections-in-intune/)

**Deployment guide:** [DEPLOYMENT.md](DEPLOYMENT.md)

---

## What's included

| Component | Purpose |
|-----------|---------|
| Logic App (`logic-app/intune-collections.logicapp.json`) | Receives the device-ID list and group name over HTTPS, looks up the Entra ID group, reconciles its membership against the list, and emails on failure |
| Power Automate flow (`power-automate-flow/`) | Runs on a schedule, queries the Power BI semantic model with a parameterized DAX query, and POSTs the result to the Logic App |

Both files are sanitized template exports. See [DEPLOYMENT.md](DEPLOYMENT.md) for the full list of placeholders to replace and a step-by-step deployment outline.

## Prerequisites

- An active **BI for Intune** deployment (the DAX queries run against its Power BI semantic model)
- An Azure subscription with permissions to create a Logic App and a managed identity
- A Power Automate environment in the same tenant
- Microsoft Graph application permissions (`Group.ReadWrite.All`, `Device.Read.All`, optionally `User.Read.All`) granted to the managed identity
- Permission to create and manage the Entra ID security group(s) the flow will populate

---

## License

[MIT](LICENSE) - use, modify, and share freely.

## Maintainer

Maintained by **PowerStacks**. Issues and pull requests welcome on the [issue tracker](../../issues).
