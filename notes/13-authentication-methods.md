# Task 13 – Authentication Methods Management

**Date:**
2026-06-26

## Goal

Query and manage user authentication methods through Microsoft Graph PowerShell SDK — list registered methods per user, generate an MFA registration report, and perform method resets.

## Connection

```powershell
Connect-MgGraph -Scopes "UserAuthenticationMethod.ReadWrite.All","User.Read.All"
```

## Key concepts

### Authentication method types

| OData type | Method | Description |
|---|---|---|
| `#microsoft.graph.passwordAuthenticationMethod` | Password | Every user has this — basic credential |
| `#microsoft.graph.phoneAuthenticationMethod` | Phone (SMS/call) | Phone number for SMS or voice call verification |
| `#microsoft.graph.microsoftAuthenticatorAuthenticationMethod` | Microsoft Authenticator | Push notification or TOTP code via the Authenticator app |
| `#microsoft.graph.fido2AuthenticationMethod` | FIDO2 security key | Hardware key (YubiKey, etc.) — passwordless |
| `#microsoft.graph.windowsHelloForBusinessAuthenticationMethod` | Windows Hello | Biometric or PIN tied to a specific device |
| `#microsoft.graph.emailAuthenticationMethod` | Email | Email address for SSPR (self-service password reset) |
| `#microsoft.graph.temporaryAccessPassAuthenticationMethod` | TAP | Time-limited passcode for onboarding or recovery |
| `#microsoft.graph.softwareOathAuthenticationMethod` | Software OATH token | Third-party TOTP apps (Google Authenticator, etc.) |

### MFA registration logic

- Every user has at least 1 method (password)
- `MethodCount > 1` means the user has registered at least one additional method beyond password
- This is the simplest proxy for "user has MFA-capable method registered"
- For more accurate reporting, check specific method types (Authenticator, FIDO2, Phone)

## Operations

| Operation | Cmdlet |
|---|---|
| List all methods | `Get-MgUserAuthenticationMethod -UserId` |
| Phone methods | `Get-MgUserAuthenticationPhoneMethod -UserId` |
| Authenticator methods | `Get-MgUserAuthenticationMicrosoftAuthenticatorMethod -UserId` |
| Password method | `Get-MgUserAuthenticationPasswordMethod -UserId` |
| Remove phone method | `Remove-MgUserAuthenticationPhoneMethod -UserId -PhoneAuthenticationMethodId` |

## MFA registration report

```powershell
$users = Get-MgUser -All -Property DisplayName,UserPrincipalName
$report = foreach ($user in $users) {
    $methods = Get-MgUserAuthenticationMethod -UserId $user.Id
    [PSCustomObject]@{
        DisplayName = $user.DisplayName
        UPN = $user.UserPrincipalName
        MethodCount = $methods.Count
        HasMFA = ($methods.Count -gt 1)
    }
}
$report | Export-Csv -Path "mfa-registration-report.csv" -NoTypeInformation
```

## What I did

- Listed authentication methods for the admin account and a lab user.
- Compared the method counts between users with and without MFA registered.
- Queried specific method types — phone, Microsoft Authenticator, password.
- Generated an MFA registration report for all users in the tenant.
- Exported the MFA report to CSV for offline analysis.
- Attempted an auth method reset (phone method removal) on a lab user.
- Disconnected the session.

## Result

Authentication methods were queried and compared across users. An MFA registration report was generated and exported, identifying which users have registered strong authentication methods.

## Lessons learned

- Every user has at least a password method — `MethodCount == 1` means the user has ONLY a password and no MFA method registered.
- `Get-MgUserAuthenticationMethod` returns generic objects with method type in `AdditionalProperties.'@odata.type'`. Use type-specific cmdlets (e.g. `Get-MgUserAuthenticationPhoneMethod`) for detailed properties.
- Removing an authentication method (e.g. phone) forces the user to re-register — this is the Graph API equivalent of "Require re-register MFA" in the portal.
- The MFA registration report loops through all users and queries methods per user — on large tenants, this is slow. For production, use the `authenticationMethodsRegistrationReport` endpoint instead.
- `UserAuthenticationMethod.ReadWrite.All` is a highly privileged scope — it allows reading AND modifying auth methods for any user. Use `UserAuthenticationMethod.Read.All` for read-only reporting.
- Phone methods have a `PhoneType` property: `mobile` (SMS/call), `alternateMobile` (backup), or `office` (desk phone). Each serves a different verification scenario.

## Evidence

```text
evidence/task-13-authentication-methods/
```
