# Task 03 – PowerShell Graph SDK Setup

**Date:**
2026-06-17

## Goal

Set up the Microsoft Graph PowerShell SDK, connect to the tenant with delegated permissions, explore available modules and cmdlets, and run basic queries.

## SDK installation

- Module: Microsoft.Graph
- Installation scope: CurrentUser
- Version: (fill after task)

## Connecting to Graph

### Delegated (interactive)

```powershell
Connect-MgGraph -Scopes "User.Read.All", "Group.Read.All"
```

- Opens browser for authentication and consent
- Scopes are requested at connection time — equivalent to permissions in Graph Explorer
- `Get-MgContext` shows: Account, TenantId, AuthType, Scopes

## Module structure

- Microsoft.Graph is split into submodules (Microsoft.Graph.Users, Microsoft.Graph.Groups, Microsoft.Graph.Identity, etc.)
- Only the needed submodule is loaded when a cmdlet is called

## Finding cmdlets

| Method | Example | Use case |
|---|---|---|
| `Get-Command -Module` | `Get-Command -Module Microsoft.Graph.Users` | Browse all cmdlets in a submodule |
| `Find-MgGraphCommand` | `Find-MgGraphCommand -Command "GET /users"` | Map a Graph API endpoint to its PowerShell cmdlet |

## SDK vs Graph Explorer syntax comparison

| Graph Explorer | PowerShell SDK |
|---|---|
| `GET /v1.0/users?$top=5&$select=displayName` | `Get-MgUser -Top 5 -Property DisplayName` |
| `GET /v1.0/users?$filter=department eq 'IT'` | `Get-MgUser -Filter "department eq 'IT'"` |
| `$orderby=displayName` | `-Sort "displayName"` or `-OrderBy "displayName"` |

## What I did

- Verified Microsoft Graph SDK installation.
- Connected to the tenant using delegated authentication with specific scopes.
- Reviewed the connection context using Get-MgContext.
- Listed available Graph submodules.
- Used Find-MgGraphCommand to map REST endpoints to PowerShell cmdlets.
- Ran basic user and group queries using SDK cmdlets.
- Disconnected the session.

## Result

The Microsoft Graph PowerShell SDK was configured and validated. Delegated authentication was used to connect, and basic tenant queries were executed using SDK cmdlets instead of raw REST calls.

## Lessons learned

- `Connect-MgGraph -Scopes` is the PowerShell equivalent of the permissions consent in Graph Explorer — you request scopes upfront.
- `Get-MgContext` is essential for troubleshooting — it shows exactly which account, tenant, and scopes are active.
- `Find-MgGraphCommand` bridges the gap between Graph API documentation (which uses REST endpoints) and the PowerShell SDK (which uses cmdlets).
- The SDK automatically handles authentication tokens, pagination headers, and retry logic — things you'd have to manage manually with raw REST calls.
- Always `Disconnect-MgGraph` when done — the token persists in memory until the session is closed or explicitly disconnected.

## Evidence

```text
evidence/task-03-powershell-graph-sdk-setup/
```
