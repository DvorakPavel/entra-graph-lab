# Entra Graph Lab — Hands-on Microsoft Graph API Automation

15 hands-on tasks. Real M365 tenant. Real Graph API calls. Real automation scripts.

I built a complete Microsoft Graph API automation toolkit from scratch — PowerShell SDK scripting, Conditional Access automation, Identity Protection workflows, security reporting, and more — documenting every API call, every SDK quirk, and every lesson learned along the way.

Every task uses the Microsoft Graph PowerShell SDK v2 against a live Entra ID P2 tenant. Every CSV is real data (redacted). The notes don't just show *what was automated* — they show *what broke* and how it was fixed.

**Related project:** [sc300-entra-lab](https://github.com/DvorakPavel/sc300-entra-lab) — 33 tasks covering the full SC-300 exam scope via Entra portal and PowerShell.

## Environment

| Component | Detail |
|---|---|
| Tenant | Microsoft 365 E5 Developer (25 licenses) |
| License | Entra ID P2 (Identity Protection, PIM) |
| SDK | Microsoft.Graph PowerShell v2.x |
| Users | 6 lab users, 2 break-glass admins, 1 admin account, sample users |
| Tools | Graph Explorer, PowerShell 7, VS Code |

## Lab Tasks

### Foundations — Graph API & PowerShell SDK

| # | Task | Focus |
|---|---|---|
| 01 | [Graph Explorer Fundamentals](notes/01-graph-explorer-fundamentals.md) | REST queries, permissions consent, response inspection |
| 02 | [Permission Model Deep Dive](notes/02-permission-model-deep-dive.md) | Delegated vs application, scopes, least privilege |
| 03 | [PowerShell Graph SDK Setup](notes/03-powershell-graph-sdk-setup.md) | Module installation, Connect-MgGraph, scope management |

### Identity Management — Users, Groups, Roles

| # | Task | Focus |
|---|---|---|
| 04 | [User CRUD & Lifecycle](notes/04-user-crud-lifecycle.md) | Create, update, disable, delete, restore users |
| 05 | [Group Management](notes/05-group-management.md) | Security vs M365 groups, dynamic membership, nesting |
| 10 | [Directory Roles & PIM](notes/10-directory-roles-pim.md) | Role templates, role assignment, PIM eligible vs permanent |

### Production Patterns — Error Handling, Pagination, Debugging

| # | Task | Focus |
|---|---|---|
| 06 | [Pagination & Throttling](notes/06-pagination-and-throttling.md) | -All flag, @odata.nextLink, 429 handling, retry logic |
| 07 | [Error Handling & Debugging](notes/07-error-handling-and-debugging.md) | Try/catch, error codes, -Debug, Graph troubleshooting |

### Security Automation — CA, Identity Protection, Named Locations

| # | Task | Focus |
|---|---|---|
| 08 | [Conditional Access Automation](notes/08-conditional-access-automation.md) | CA policy CRUD, grant controls, conditions, report-only |
| 09 | [Identity Protection API](notes/09-identity-protection-api.md) | Risky users, risk detections, confirm/dismiss, bulk ops |
| 14 | [Named Locations & Network Conditions](notes/14-named-locations.md) | IP-based, country-based, IsTrusted, CA policy integration |

### Monitoring & Reporting — Logs, Auth Methods, Apps, Security Report

| # | Task | Focus |
|---|---|---|
| 11 | [Audit & Sign-in Log Analysis](notes/11-audit-signin-log-analysis.md) | Sign-in logs, audit logs, error codes, CSV export, stale accounts |
| 12 | [App Registration & Service Principal](notes/12-app-registration-service-principal.md) | App vs SP, client secrets, API permissions, lifecycle |
| 13 | [Authentication Methods](notes/13-authentication-methods.md) | MFA registration report, method types, auth method reset |
| 15 | [Automated Security Report](notes/15-automated-security-report.md) | Multi-endpoint CSV report, HTML dashboard, stale account detection |

## Capstone: Automated Tenant Security Report (Task 15)

The final task pulls data from 6 Graph API endpoints into a single security dashboard:

| Metric | Value | Status |
|---|---|---|
| Total Users | 27 | — |
| Users with MFA | 27 | All users have MFA |
| Risky Users | 4 | All dismissed (from previous testing) |
| CA Policies | 11 | 8 enforced, 3 report-only |
| App Secrets Expiring (30d) | 0 | OK |
| Stale Accounts (90+ days) | 16 | Review needed (demo tenant users) |

## Repository Structure

```
notes/      # Detailed notes per task (goal, concepts, commands, lessons learned)
evidence/   # Screenshots and redacted CSV exports organized by task
README.md
```

## Key Takeaways

Things I learned the hard way — not from docs, but from doing:

- **CA policies auto-remediate Identity Protection risk** — Confirming a user as compromised via `Confirm-MgRiskyUserCompromised` triggers CA-Risk-User-Force-Password-Change immediately. The risk state shows "remediated" before you can even observe "confirmedCompromised". Switch the CA policy to report-only before testing.
- **`New-MgDirectoryRoleMember` doesn't exist in SDK v2** — The correct cmdlet is `New-MgDirectoryRoleMemberByRef` with an `@odata.id` body parameter. SDK v2 renamed many cmdlets and the old documentation is misleading.
- **Trusted named locations can't be deleted** — You must first set `IsTrusted = $false` via `Update-MgIdentityConditionalAccessNamedLocation`, then delete. The API returns a 400 error with no obvious fix if you skip this step.
- **`SecretText` is shown exactly once** — When you create a client secret with `Add-MgApplicationPassword`, the actual secret value is only in the response. Query the secret later and you get metadata only. This is by design, not a bug.
- **Risk propagation on dev tenants is slow** — `Invoke-MgDismissRiskyUser` can take 60+ seconds to reflect in `Get-MgRiskyUser`. Always add `Start-Sleep` and retry logic — don't assume the operation failed.
- **Non-terminating errors are silent killers** — `Remove-MgUserAuthenticationPhoneMethod` can fail but your script continues and prints "success". Always use `-ErrorAction Stop` or check `$?` after critical operations.
- **`SignInActivity` requires explicit request** — `Get-MgUser -All` does NOT return last sign-in date. You must add `-Property SignInActivity` explicitly, and it requires Entra ID P1.
- **Updating IP ranges replaces the entire array** — When updating a named location, you must include ALL existing CIDR ranges plus the new ones. The API replaces, it doesn't append.

## Tools & Technologies

Microsoft Graph API, Microsoft Graph PowerShell SDK v2, Graph Explorer, PowerShell 7, Entra ID P2, Conditional Access, Identity Protection, PIM, Named Locations, Application Registrations, Service Principals, Authentication Methods, Sign-in & Audit Logs

## Author

**Pavel Dvořák** — [LinkedIn](https://www.linkedin.com/in/pavel-dvorak88) | [GitHub](https://github.com/DvorakPavel)
