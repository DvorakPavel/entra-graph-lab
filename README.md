# entra-graph-lab

Hands-on Microsoft Graph API lab — 15 tasks covering permissions, PowerShell automation, Conditional Access, Identity Protection, and more. Built on a real Microsoft 365 developer tenant with Entra ID P2.

## About

This repository documents my hands-on learning with the Microsoft Graph API as part of the SC-300 (Microsoft Identity and Access Administrator) exam preparation. Each task explores a different area of Entra ID through Graph Explorer and PowerShell Graph SDK, with detailed notes and screenshot evidence.

**Related project:** [sc300-entra-lab](https://github.com/DvorakPavel/sc300-entra-lab) — 33 tasks covering the full SC-300 exam scope via Entra portal and PowerShell.

## Lab environment

| Component | Detail |
|---|---|
| Tenant | Microsoft 365 E5 Developer (25 licenses) |
| License | Entra ID P2 (Identity Protection, PIM) |
| SDK | Microsoft.Graph PowerShell v2.x |
| Tools | Graph Explorer, PowerShell 7, VS Code |

## Task overview

| # | Task | Focus area |
|---|---|---|
| 01 | [Graph Explorer Fundamentals](notes/01-graph-explorer-fundamentals.md) | REST queries, permissions consent, response inspection |
| 02 | [Permission Model Deep Dive](notes/02-permission-model-deep-dive.md) | Delegated vs application, scopes, least privilege |
| 03 | [PowerShell Graph SDK Setup](notes/03-powershell-graph-sdk-setup.md) | Module installation, Connect-MgGraph, scope management |
| 04 | [User CRUD & Lifecycle](notes/04-user-crud-lifecycle.md) | Create, update, disable, delete, restore users |
| 05 | [Group Management](notes/05-group-management.md) | Security vs M365 groups, dynamic membership, nesting |
| 06 | [Pagination & Throttling](notes/06-pagination-and-throttling.md) | -All flag, @odata.nextLink, 429 handling, retry logic |
| 07 | [Error Handling & Debugging](notes/07-error-handling-and-debugging.md) | Try/catch, error codes, -Debug, Graph troubleshooting |
| 08 | [Conditional Access Automation](notes/08-conditional-access-automation.md) | CA policy CRUD, grant controls, conditions, report-only |
| 09 | [Identity Protection API](notes/09-identity-protection-api.md) | Risky users, risk detections, confirm/dismiss, bulk ops |
| 10 | [Directory Roles & PIM](notes/10-directory-roles-pim.md) | Role templates, role assignment, PIM eligible vs permanent |
| 11 | [Audit & Sign-in Log Analysis](notes/11-audit-signin-log-analysis.md) | Sign-in logs, audit logs, error codes, CSV export, stale accounts |
| 12 | [App Registration & Service Principal](notes/12-app-registration-service-principal.md) | App vs SP, client secrets, API permissions, lifecycle |
| 13 | [Authentication Methods](notes/13-authentication-methods.md) | MFA registration report, method types, auth method reset |
| 14 | [Named Locations & Network Conditions](notes/14-named-locations.md) | IP-based, country-based, IsTrusted, CA policy integration |
| 15 | [Automated Security Report](notes/15-automated-security-report.md) | Multi-endpoint CSV report, HTML dashboard, stale account detection |

## Repository structure

```
entra-graph-lab/
├── notes/                          # Detailed notes per task
│   ├── 01-graph-explorer-fundamentals.md
│   ├── ...
│   └── 15-automated-security-report.md
├── evidence/                       # Screenshots and redacted CSV exports
│   ├── task-01-graph-explorer-fundamentals/
│   ├── ...
│   └── task-15-security-report/
└── README.md
```

## What each note contains

Every task note follows a consistent structure:

- **Goal** — what the task set out to achieve
- **Connection** — exact `Connect-MgGraph` scopes used
- **Key concepts** — theory and reference tables relevant to the task
- **Operations** — cmdlet reference table
- **What I did** — step-by-step actions performed
- **Result** — outcome summary
- **Lessons learned** — real issues encountered and how they were resolved

## Key skills demonstrated

**Graph API & PowerShell automation** — all 15 tasks use Microsoft Graph PowerShell SDK v2 with explicit commands, proper error handling, and production-ready patterns.

**Security operations** — Identity Protection risk management, MFA registration auditing, stale account detection, app secret expiry monitoring, CA policy automation.

**Troubleshooting & debugging** — SDK v2 cmdlet naming changes (e.g. `New-MgDirectoryRoleMemberByRef`), CA policy interference with Identity Protection testing, non-terminating error handling, propagation delays.

**Reporting & export** — CSV exports with proper redaction for public sharing, HTML dashboard generation, and multi-endpoint data aggregation.

## Highlights from the lab

- **Task 09**: Discovered that a CA policy (CA-Risk-User-Force-Password-Change) auto-remediated risk before it could be observed — had to switch to report-only for testing.
- **Task 10**: Found that `New-MgDirectoryRoleMember` doesn't exist in SDK v2 — the correct cmdlet is `New-MgDirectoryRoleMemberByRef` with an `@odata.id` body.
- **Task 14**: Learned that trusted named locations cannot be deleted — you must set `IsTrusted = $false` first, then delete.
- **Task 15**: Built a capstone security report pulling from 6 Graph API endpoints, identifying 16 stale accounts and monitoring app secret expiry.


## Author

**Pavel Dvorak**
- GitHub: [@DvorakPavel](https://github.com/DvorakPavel)
- LinkedIn: [pavel-dvorak88](https://www.linkedin.com/in/pavel-dvorak88/)
