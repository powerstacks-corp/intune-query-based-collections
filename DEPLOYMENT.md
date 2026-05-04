# Deployment Guide

This repo contains two artifacts you can adapt to your own tenant to implement query-based device collections for Microsoft Intune.

## Contents

```
logic-app/
  intune-collections.logicapp.json     <- ARM template export of the Logic App
power-automate-flow/
  manifest.json
  Microsoft.Flow/
    flows/
      manifest.json
      <flow-id>/
        apisMap.json
        connectionsMap.json
        definition.json                <- the actual Power Automate flow
```

Both files are **template exports** sanitized of customer-specific data. Before deploying, you will need to substitute placeholders for values from your own Azure subscription, Power BI tenant, and Entra ID.

## Placeholders to replace

| Placeholder | What to substitute |
|-------------|--------------------|
| `00000000-0000-0000-0000-000000000000` | Your Azure subscription ID |
| `<your-resource-group>` | The resource group where the Logic App and managed identity live |
| `<your-tenant>.onmicrosoft.com` | Your Entra ID tenant primary domain (in portal URLs inside email bodies) |
| `<sender@your-domain.com>` | The from-address for failure notification emails |
| `<recipient@your-domain.com>` | The recipient address for failure notification emails |
| `<replace-with-your-logic-app-trigger-url>` | The HTTPS POST URL of your Logic App's "When a HTTP request is received" trigger (you copy this from the Logic App after the first save) |
| `Sample App` | The app name (or other identifier) that your DAX query filters on |
| `Sample Category A`, `Sample Category B` | Replace with your Intune device categories, or remove the second filter table if you don't filter by category |

## Prerequisites

- An Azure subscription with permissions to create:
  - A Logic App (Standard or Consumption plan)
  - A user-assigned managed identity (or you can use the Logic App's system-assigned identity)
- The managed identity must have the Microsoft Graph application permissions:
  - `Group.ReadWrite.All` (to add and remove group members)
  - `Device.Read.All` (to look up device IDs)
  - `User.Read.All` (only if you also want to manage user-targeted groups)
- A Power Automate environment in the same tenant
- A Power BI workspace running BI for Intune (or another semantic model that exposes the device IDs you want to query)

## Deployment outline

1. **Deploy the Logic App.**
   1. In the Azure portal, create a new Logic App resource.
   1. Open it in code view and paste the contents of `logic-app/intune-collections.logicapp.json` (after substituting placeholders).
   1. Save and copy the HTTPS POST URL from the "When a HTTP request is received" trigger. This URL contains a SAS signature; treat it as a secret.
1. **Grant managed-identity permissions.**
   1. Identify the managed identity used by the Logic App.
   1. In Entra ID, grant the Microsoft Graph application permissions listed above.
   1. Admin consent the permissions.
1. **Import the Power Automate flow.**
   1. In Power Automate, choose **Import** and upload the `PatchGroupAutomationFlow_<timestamp>.zip` you would create from the `power-automate-flow/` directory contents (or import the directory directly if your tooling supports it).
   1. During import, map the connection references to your tenant's connectors (Power BI, Office 365 Outlook).
   1. Open the imported flow and replace `<replace-with-your-logic-app-trigger-url>` in the HTTP action with the Logic App URL from step 1.
1. **Adapt the DAX query.**
   1. The flow contains a sample DAX query that returns Entra ID device IDs for an example app and category. Replace the filter table contents (`Sample App`, `Sample Category A`, `Sample Category B`) with the values that match your selection criteria.
   1. Test the query in Power BI Desktop first to confirm it returns the expected devices.
1. **Pick a schedule.**
   1. The flow's recurrence trigger defaults to a sample schedule. Adjust to your operational cadence (daily is typical; nightly is a safe starting point).
1. **Validate.**
   1. Trigger the flow manually.
   1. Confirm the Logic App run history shows a successful invocation.
   1. Confirm the target Entra ID group's membership now matches the DAX query result.

## Security notes

- The SAS-signed Logic App invoke URL is the only credential the Power Automate flow needs to call the Logic App. Treat it like a password: do not check it into source control, do not log it, rotate it if it leaks.
- The Logic App uses a managed identity to call Microsoft Graph; no client secrets are stored in either artifact.
- Failure-notification emails are sent through the Office 365 Outlook connector running under whichever account you authorize the connector with.

## Companion blog post

[How to create query-based collections in Intune](https://powerstacks.com/blog/how-to-create-query-based-collections-in-intune/) walks through the architecture and decisions behind this design.
