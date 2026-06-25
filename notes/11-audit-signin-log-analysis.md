# Task 11 – Audit & Sign-in Log Analysis

**Date:**
2026-06-25

## Goal

Query and analyse sign-in logs and directory audit logs through Microsoft Graph PowerShell SDK — filter by status, user, category, and export results to CSV for reporting.

## Connection

```powershell
Connect-MgGraph -Scopes "AuditLog.Read.All","Directory.Read.All"
```

## Key concepts

### Sign-in logs vs Audit logs

| Log type | What it captures | Use case |
|---|---|---|
| Sign-in log | Authentication events — who signed in, from where, success/failure, CA evaluation | Investigating access issues, detecting compromised accounts |
| Audit log | Directory changes — user created, role assigned, policy updated, group modified | Change tracking, compliance, forensics |

### Important sign-in log properties

| Property | Description |
|---|---|
| Status.ErrorCode | 0 = success, non-zero = failure (e.g. 50126 = bad password, 53003 = blocked by CA) |
| ConditionalAccessStatus | success, failure, notApplied — shows whether CA policies were evaluated and enforced |
| IPAddress | Source IP of the sign-in attempt |
| AppDisplayName | Which application the user signed into |
| IsInteractive | True = user-driven sign-in, False = token refresh or service call |

### Common error codes

| Error code | Meaning |
|---|---|
| 0 | Success |
| 50126 | Invalid username or password |
| 50076 | MFA required but not completed |
| 53003 | Blocked by Conditional Access policy |
| 50053 | Account locked (too many failed attempts) |
| 50057 | Account disabled |

## Operations

| Operation | Cmdlet |
|---|---|
| Sign-in logs | `Get-MgAuditLogSignIn` |
| Audit logs | `Get-MgAuditLogDirectoryAudit` |
| Filter by error | `-Filter "status/errorCode ne 0"` |
| Filter by user | `-Filter "userPrincipalName eq '...'"` |
| Filter by category | `-Filter "category eq 'RoleManagement'"` |
| Export to CSV | `Export-Csv -Path "report.csv" -NoTypeInformation` |

## Stale accounts

A stale account is a user who hasn't signed in for an extended period (typically 90+ days). These represent a security risk because they could be compromised without anyone noticing. Identifying and disabling stale accounts is a key part of tenant hygiene.

```powershell
Get-MgUser -All -Property DisplayName,UserPrincipalName,SignInActivity | Where-Object {
    $_.SignInActivity.LastSignInDateTime -lt (Get-Date).AddDays(-90)
}
```

Note: The `SignInActivity` property requires at least an Entra ID P1 license.

## What I did

- Queried the 10 most recent sign-in events.
- Filtered sign-in logs for failed authentications (errorCode ne 0).
- Filtered sign-in logs for a specific user to review their sign-in history.
- Queried directory audit logs for recent changes.
- Filtered audit logs by category (RoleManagement, Policy) to review specific change types.
- Exported all failed sign-ins to a CSV file for offline analysis.
- Identified stale accounts that haven't signed in for 90+ days.
- Disconnected the session.

## Result

Sign-in and audit logs were queried, filtered, and exported through the Graph PowerShell SDK. Failed sign-in analysis and stale account detection were demonstrated as practical security monitoring tasks.

## Lessons learned

- Sign-in logs capture authentication events (who tried to access what), and audit logs capture directory changes (who changed what). Both are essential for security monitoring.
- `Status.ErrorCode` is the key field for filtering failed sign-ins — 0 means success, anything else is a failure with a specific reason code.
- `ConditionalAccessStatus` in sign-in logs shows whether CA policies were applied — this is invaluable for troubleshooting "why can't I sign in" issues.
- `-All` on sign-in logs can return massive datasets in production — always use `-Filter` and `-Top` to scope the query, or use date range filters.
- `Export-Csv -NoTypeInformation` removes the PowerShell type header line that would otherwise appear as the first row — always use this flag for clean CSV output.
- `SignInActivity` (last sign-in date per user) requires Entra ID P1 and must be explicitly requested via `-Property` — it's not returned by default.
- Stale account detection is a recurring security task — the script from this task can be scheduled to run weekly and flag accounts for review.

## Evidence

```text
evidence/task-11-audit-signin-log-analysis/
```
