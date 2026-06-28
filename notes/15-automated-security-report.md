# Task 15 – Automated Tenant Security Report

**Date:**
2026-06-28

## Goal

Build an automated security report that pulls data from multiple Microsoft Graph API endpoints, generates individual CSV reports, and produces an HTML summary dashboard — a capstone task combining skills from all previous tasks.

## Connection

```powershell
Connect-MgGraph -Scopes "User.Read.All","UserAuthenticationMethod.Read.All","IdentityRiskEvent.Read.All","IdentityRiskyUser.Read.All","Policy.Read.All","Application.Read.All","AuditLog.Read.All","Directory.Read.All"
```

## Key concepts

### Multi-endpoint reporting

A production security report should aggregate data from multiple Graph API endpoints to give a holistic view of tenant security posture. Each endpoint answers a different question:

| Endpoint | Question answered |
|---|---|
| `Get-MgUser` + `Get-MgUserAuthenticationMethod` | Which users have MFA registered? |
| `Get-MgRiskyUser` | Are there users with active or dismissed risk? |
| `Get-MgIdentityConditionalAccessPolicy` | What CA policies exist, and are they enforced? |
| `Get-MgApplication` (PasswordCredentials) | Are any app secrets expired or expiring soon? |
| `Get-MgIdentityConditionalAccessNamedLocation` | What named locations are configured? |
| `SignInActivity` property on users | Are there stale accounts that haven't signed in? |

### Stale account detection

- `SignInActivity` requires Entra ID P1 license
- Property is on the user object but must be explicitly requested via `-Property SignInActivity`
- A common threshold is 90 days — accounts inactive longer than that should be reviewed for disabling or deletion
- Demo/test tenant users created by the Microsoft 365 developer program often show as stale because they only sign in once at provisioning

### HTML summary dashboard

- PowerShell here-strings (`@"..."@`) are ideal for generating HTML with embedded PowerShell expressions
- Colour-coded status (green for OK, orange/red for warnings) makes the report scannable at a glance
- The summary should highlight actionable items — things an admin needs to review or fix

## Reports generated

| # | Report | Records | File |
|---|---|---|---|
| 01 | User Security Overview | 27 | `01-user-security-overview.csv` |
| 02 | Risky Users | 4 | `02-risky-users.csv` |
| 03 | Conditional Access Policies | 11 | `03-conditional-access-policies.csv` |
| 04 | App Secret Expiry | 1 | `04-app-secret-expiry.csv` |
| 05 | Named Locations | 2 | `05-named-locations.csv` |
| 06 | Stale Accounts (90+ days) | 16 | `06-stale-accounts.csv` |
| 00 | HTML Summary Dashboard | — | `00-security-report-summary.html` |

## What I did

- Connected to Microsoft Graph with scopes covering users, authentication methods, risk, policies, applications, and audit logs.
- Generated a user security overview report with MFA status, last sign-in date, and days since last sign-in for all 27 tenant users.
- Exported all risky users with their risk level, state, and detail — 4 users with dismissed risk from previous Identity Protection testing.
- Listed all 11 Conditional Access policies with state, grant controls, and user scope — 8 enabled, 3 in report-only mode.
- Checked all application registrations for client secret expiry — 1 secret found, status OK (116 days until expiry).
- Exported named locations — 2 remaining (LAB-Trusted-Office-Network and LAB-Blocked-Countries from SC-300 lab).
- Identified 16 stale accounts with no sign-in activity in 90+ days (mostly Microsoft 365 demo users provisioned at tenant creation).
- Generated an HTML summary dashboard with colour-coded status indicators for each metric.
- Created redacted versions of all CSV files for public GitHub evidence.
- Disconnected the session.

## Summary findings

| Metric | Value | Status |
|---|---|---|
| Total Users | 27 | — |
| Users with MFA | 27 | All users have MFA |
| Risky Users | 4 | ACTION REQUIRED (all dismissed, from previous testing) |
| CA Policies | 11 | 8 enabled |
| App Secrets Expiring (30d) | 0 | OK |
| Stale Accounts (90+ days) | 16 | REVIEW NEEDED |

## Result

A comprehensive tenant security report was generated from 6 different Graph API endpoints, exported to individual CSV files, and summarised in an HTML dashboard. The report identified 16 stale accounts and 4 previously dismissed risky users as items requiring review.

## Lessons learned

- Combining multiple Graph API endpoints into one report requires careful scope planning — missing a scope causes the entire connection to fail or specific queries to return empty results.
- `SignInActivity` is not returned by default on `Get-MgUser` — it must be explicitly requested via `-Property SignInActivity`. Forgetting this produces `$null` for LastSignIn on every user.
- The `$userReport` variable from the user overview step can be reused for the stale accounts report — no need to query the API again. This is a practical example of storing intermediate results to reduce API calls.
- PowerShell here-strings (`@"..."@`) with embedded expressions (`$()`) are a clean way to generate HTML reports without external templating libraries.
- Demo tenant users (Adele Vance, Alex Wilber, etc.) created by the Microsoft 365 developer program sign in only once at provisioning and then appear as stale — in a production environment, these would be real accounts that need review.
- For production use, this script should be scheduled (e.g., Azure Automation runbook or Task Scheduler) and the HTML report emailed to the security team.
- Redacting CSV files for public sharing requires replacing tenant domain, Object IDs, App IDs, and guest user email addresses — a consistent redaction pattern (`******` for tenant, `REDACTED-*` for IDs) makes the evidence readable while protecting sensitive data.

## Evidence

```text
evidence/task-15-security-report/
```
