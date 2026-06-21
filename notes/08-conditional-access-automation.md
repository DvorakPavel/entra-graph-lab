# Task 08 – Conditional Access Automation

**Date:**
2026-06-21

## Goal

Manage Conditional Access policies programmatically through Microsoft Graph PowerShell SDK — read, backup, create, update, and delete policies.

## Connection

```powershell
Connect-MgGraph -Scopes "Policy.Read.All","Policy.ReadWrite.ConditionalAccess","Application.Read.All"
```

## CA policy JSON structure

```json
{
    "displayName": "Policy name",
    "state": "enabled | disabled | enabledForReportingButNotEnforced",
    "conditions": {
        "users": {
            "includeUsers": ["All"],
            "excludeUsers": ["breakglass-id-1", "breakglass-id-2"]
        },
        "applications": {
            "includeApplications": ["All"]
        },
        "clientAppTypes": ["browser", "mobileAppsAndDesktopClients"]
    },
    "grantControls": {
        "operator": "OR",
        "builtInControls": ["mfa"]
    },
    "sessionControls": { }
}
```

Key sections: `conditions` (who, what, where, which apps), `grantControls` (what to enforce), `sessionControls` (session behaviour).

## Operations

| Operation | Cmdlet | Permission |
|---|---|---|
| List all | `Get-MgIdentityConditionalAccessPolicy` | Policy.Read.All |
| Read detail | `Get-MgIdentityConditionalAccessPolicy -Id` | Policy.Read.All |
| Create | `New-MgIdentityConditionalAccessPolicy -BodyParameter` | Policy.ReadWrite.ConditionalAccess |
| Update | `Update-MgIdentityConditionalAccessPolicy -Id` | Policy.ReadWrite.ConditionalAccess |
| Delete | `Remove-MgIdentityConditionalAccessPolicy -Id` | Policy.ReadWrite.ConditionalAccess |

## Backup

```powershell
$allPolicies = Get-MgIdentityConditionalAccessPolicy
$allPolicies | ConvertTo-Json -Depth 10 | Out-File "ca-policies-backup.json"
```

- `-Depth 10` is essential — without sufficient depth, nested objects (Conditions, GrantControls) are truncated
- Backup JSON can be used for documentation, disaster recovery, or migrating policies between tenants

## Create policy via Graph

- Always create in `enabledForReportingButNotEnforced` (report-only) first
- Test in report-only, verify in sign-in logs, then switch to `enabled`
- `BuiltInControls` options: `mfa`, `block`, `compliantDevice`, `domainJoinedDevice`, `passwordChange`

## What I did

- Listed all existing Conditional Access policies.
- Retrieved the full JSON detail of a specific policy to understand the structure.
- Exported all policies to a backup JSON file.
- Created a test CA policy (block legacy auth) in report-only mode via Graph.
- Verified the policy appeared in the Entra admin center.
- Updated the policy name via Graph.
- Deleted the test policy and confirmed removal.
- Disconnected the session.

## Result

The full Conditional Access policy lifecycle was managed through the Graph PowerShell SDK. Policies were read, backed up, created, modified, and removed without using the Entra portal.

## Lessons learned

- `ConvertTo-Json -Depth 10` is critical for CA policies — the default depth (2) truncates nested objects like Conditions and GrantControls, producing an incomplete backup.
- Always create CA policies in report-only (`enabledForReportingButNotEnforced`) via Graph — a script that creates enforced policies can lock out users instantly.
- The JSON structure of a CA policy is the same in Graph Explorer and PowerShell — learning the structure once applies everywhere.
- `Policy.Read.All` is enough for reading and backup. `Policy.ReadWrite.ConditionalAccess` is needed for create/update/delete — separate the scopes based on the task.
- Backup + restore via JSON is a practical disaster recovery strategy — if a policy is accidentally deleted, it can be recreated from the export.

## Evidence

```text
evidence/task-08-conditional-access-automation/
```
