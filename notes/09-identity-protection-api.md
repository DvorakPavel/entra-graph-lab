# Task 09 – Identity Protection API

**Date:**
2026-06-21

## Goal

Manage Identity Protection programmatically through Microsoft Graph — query risky users and risk detections, confirm compromised users, dismiss risk, and perform bulk operations.

## Connection

```powershell
Connect-MgGraph -Scopes "IdentityRiskyUser.ReadWrite.All","IdentityRiskEvent.Read.All"
```

## Key concepts

### RiskLevel vs RiskState

| Property | Values | Meaning |
|---|---|---|
| RiskLevel | none, low, medium, high, hidden | Severity of the detected risk |
| RiskState | atRisk, confirmedCompromised, dismissed, remediated, confirmedSafe | Current status of the risk investigation |

### Risk event types

| Type | Source |
|---|---|
| unfamiliarFeatures | Automated detection — unusual sign-in properties |
| anonymizedIPAddress | Automated detection — sign-in from anonymising service |
| adminConfirmedUserCompromised | Manual — admin confirmed via portal or API |
| leakedCredentials | Automated — credentials found in public data breach |

## Operations

| Operation | Cmdlet | Purpose |
|---|---|---|
| List risky users | `Get-MgRiskyUser` | View all users with active risk |
| List risk detections | `Get-MgRiskDetection` | View all risk events |
| Confirm compromised | `Confirm-MgRiskyUserCompromised -UserIds` | Mark user as compromised (sets risk to High) |
| Dismiss risk | `Invoke-MgDismissRiskyUser -UserIds` | Close risk without remediation |
| Bulk operations | Pass array of UserIds | Process multiple users in one call |

## Interaction with Conditional Access

- Confirming a user as compromised sets their RiskLevel to High
- If a CA policy targets user risk (e.g. CA-Risk-User-Force-Password-Change), it triggers immediately
- Dismissing risk removes the CA enforcement for that user
- This creates a complete investigate → respond → resolve workflow

## What I did

- Queried the risky users list and risk detections.
- Confirmed a lab user as compromised via Graph API and verified the risk state change.
- Dismissed the user risk and verified the state change.
- Performed bulk confirm compromised on multiple users simultaneously.
- Performed bulk dismiss to clean up all confirmed risks.
- Reviewed risk detections to see the adminConfirmedUserCompromised events created by the API calls.
- Disconnected the session.

## Result

Identity Protection risk management was fully automated through the Graph API. Single-user and bulk operations were validated for both confirm and dismiss workflows.

## Lessons learned

- `Confirm-MgRiskyUserCompromised` immediately sets RiskLevel to High — if a user risk CA policy is active (e.g. CA-Risk-User-Force-Password-Change), the risk is auto-remediated and dismissed before you can even observe it. Disable or switch the CA policy to report-only first when testing.
- `Invoke-MgDismissRiskyUser` clears the risk without requiring the user to remediate (e.g. password change). Use only after investigation confirms a false positive.
- Bulk operations accept an array of UserIds — essential for incident response when multiple accounts are compromised simultaneously.
- Risk state propagation on a dev tenant is slow — confirm and dismiss operations can take 30–60+ seconds to reflect in `Get-MgRiskyUser` queries. Always add `Start-Sleep` and retry logic in scripts.
- Every `Confirm-MgRiskyUserCompromised` call creates a risk detection of type `adminConfirmedUserCompromised` — this provides an audit trail of manual risk actions.
- `Get-MgRiskyUser` only returns users with active or historical risk — users who have never had a risk event do not appear in the list.
- The Graph API risk workflow (confirm → investigate → dismiss/remediate) mirrors the portal experience but enables automation and integration with SIEM/SOAR tools.

## Evidence

```text
evidence/task-09-identity-protection-api/
```
