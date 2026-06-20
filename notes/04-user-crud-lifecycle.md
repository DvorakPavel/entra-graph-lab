# Task 04 – User CRUD & Lifecycle

**Date:**
2026-06-20

## Goal

Manage the full user lifecycle through Microsoft Graph PowerShell SDK — create, read, update, disable, delete, and restore a user account.

## Connection

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All"
```

## CRUD operations

| Operation | Cmdlet | Permission required |
|---|---|---|
| Create | `New-MgUser` | User.ReadWrite.All |
| Read | `Get-MgUser` | User.Read.All |
| Update | `Update-MgUser` | User.ReadWrite.All |
| Delete | `Remove-MgUser` | User.ReadWrite.All |
| Restore | `Restore-MgDirectoryDeletedItem` | User.ReadWrite.All |

## Create

```powershell
$passwordProfile = @{
    Password = "TempPass123!"
    ForceChangePasswordNextSignIn = $true
}

New-MgUser -DisplayName "Graph Test User" `
    -UserPrincipalName "graph-test-user@xxxxxx.onmicrosoft.com" `
    -MailNickname "graph-test-user" `
    -AccountEnabled:$true `
    -PasswordProfile $passwordProfile `
    -Department "IT" `
    -JobTitle "Graph API Tester"
```

- `ForceChangePasswordNextSignIn` ensures the user sets their own password on first login
- `MailNickname` is required even though it's not visible in the portal

## Update

```powershell
Update-MgUser -UserId "graph-test-user@xxxxxx.onmicrosoft.com" -JobTitle "Senior Graph API Tester" -Department "Engineering"
```

- Update-MgUser has no output on success — verify with Get-MgUser after

## Disable

```powershell
Update-MgUser -UserId "graph-test-user@xxxxxx.onmicrosoft.com" -AccountEnabled:$false
```

- Disabling blocks sign-in immediately, but does not revoke active sessions
- To fully cut access: disable + revoke sessions

## Delete and Restore

- `Remove-MgUser` performs a **soft delete** — user moves to Deleted users
- Soft-deleted users remain recoverable for **30 days**
- `Restore-MgDirectoryDeletedItem` recovers the user with all properties intact
- After 30 days, permanent deletion occurs automatically

## What I did

- Connected to Graph with User.ReadWrite.All scope.
- Created a test user with a password profile and department assignment.
- Read the user back to verify all properties.
- Updated JobTitle and Department properties.
- Disabled the account by setting AccountEnabled to false.
- Deleted the user and confirmed soft-delete in Deleted users.
- Restored the deleted user and verified recovery.
- Cleaned up by deleting the test user again.
- Disconnected the session.

## Result

The full user lifecycle was managed entirely through the Microsoft Graph PowerShell SDK. All CRUD operations were validated, including soft delete and restore.

## Lessons learned

- `New-MgUser` requires `-MailNickname` even though the portal doesn't surface it — without it, the command fails.
- `Update-MgUser` produces no output on success — always verify changes with a follow-up `Get-MgUser`.
- `Remove-MgUser` is a soft delete with a 30-day recovery window. There is no single-step hard delete via Graph — permanent deletion happens automatically after 30 days.
- Disabling an account (`AccountEnabled:$false`) blocks new sign-ins but does not terminate existing sessions. For immediate revocation, combine with `Revoke-MgUserSignInSession`.
- `ForceChangePasswordNextSignIn` is a security baseline for programmatically created accounts — the admin never needs to know the user's real password.

## Evidence

```text
evidence/task-04-user-crud-lifecycle/
```
