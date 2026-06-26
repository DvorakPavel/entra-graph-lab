# Task 12 – App Registration & Service Principal Management

**Date:**
2026-06-26

## Goal

Create and manage application registrations and service principals through Microsoft Graph PowerShell SDK — understand the relationship between apps and service principals, manage credentials, and assign API permissions.

## Connection

```powershell
Connect-MgGraph -Scopes "Application.ReadWrite.All","AppRoleAssignment.ReadWrite.All"
```

## Key concepts

### App Registration vs Service Principal

| Concept | Description | Analogy |
|---|---|---|
| App Registration | The application definition — name, permissions, redirect URIs, credentials | Blueprint of a house |
| Service Principal | The instance of the app in a specific tenant — what actually authenticates and gets permissions | The built house you can live in |

- An app registration can exist without a service principal (just a definition, can't sign in)
- A service principal is automatically created when an app is used in a tenant for the first time
- Multi-tenant apps have one app registration, but one service principal per tenant that uses them

### Permission types in RequiredResourceAccess

| Type | Name | Meaning |
|---|---|---|
| `Scope` | Delegated permission | Acts on behalf of a signed-in user — limited to what the user can access |
| `Role` | Application permission | Acts as the app itself — no user context, broader access, requires admin consent |

### Client credentials

| Credential type | Use case | Security |
|---|---|---|
| Client secret (password) | Simple to set up, shared secret | Rotate regularly, expires automatically |
| Certificate | More secure, asymmetric cryptography | Preferred for production |
| Federated credential | Workload identity federation — no stored secret | Best security, used with managed identities |

## Operations

| Operation | Cmdlet |
|---|---|
| List apps | `Get-MgApplication` |
| Create app | `New-MgApplication` |
| Create service principal | `New-MgServicePrincipal -AppId` |
| Add client secret | `Add-MgApplicationPassword -ApplicationId` |
| Add API permission | `Update-MgApplication -RequiredResourceAccess` |
| Delete app | `Remove-MgApplication -ApplicationId` |

## What I did

- Listed all existing application registrations in the tenant.
- Created a test application registration (Graph-Lab-Test-App).
- Created a service principal for the app to enable authentication.
- Added a client secret with a 6-month expiration.
- Verified that SecretText is only visible at creation time — subsequent queries show only metadata.
- Assigned a delegated API permission (User.Read) to the app.
- Compared the App Registration and Service Principal objects — same AppId, different Object IDs.
- Cleaned up by deleting the test application.
- Disconnected the session.

## Result

The full application lifecycle was managed through the Graph PowerShell SDK — creation, credential management, permission assignment, and deletion. The relationship between App Registration and Service Principal was demonstrated.

## Lessons learned

- App Registration and Service Principal (SP) share the same `AppId` but have different Object IDs (`Id`). The App Registration ID is used for `Get-MgApplication`, the Service Principal ID for `Get-MgServicePrincipal`. Confusing the two is a common source of errors.
- `Add-MgApplicationPassword` returns `SecretText` only once — if you don't capture it at creation time, you must create a new secret. This is a security feature, not a bug.
- `Type: Scope` = delegated permission (acts on behalf of a user), `Type: Role` = application permission (acts as the app itself). Using the wrong type is a frequent mistake.
- Client secrets should always have an expiration date — secrets without expiry are a security risk. Microsoft recommends a maximum of 24 months, but shorter is better.
- Creating an App Registration does NOT automatically create a Service Principal — you must call `New-MgServicePrincipal` separately. Without an SP, the app cannot authenticate.
- Deleting an App Registration automatically removes its Service Principal — but deleting just the Service Principal leaves the App Registration intact.

## Evidence

```text
evidence/task-12-app-registration-service-principal/
```
