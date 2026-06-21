# Task 07 – Error Handling & Debugging

**Date:**
2026-06-21

## Goal

Understand how Microsoft Graph API communicates errors, how to handle them in PowerShell scripts, and how to use debugging tools for troubleshooting.

## HTTP status codes

| Code | Meaning | Common cause |
|---|---|---|
| 200 | OK | Request succeeded |
| 201 | Created | Resource created (POST) |
| 204 | No Content | Update/delete succeeded (no response body) |
| 400 | Bad Request | Invalid query syntax, bad filter, missing required property |
| 401 | Unauthorized | Token expired or missing |
| 403 | Forbidden | Valid token but insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Resource already exists or version conflict |
| 429 | Too Many Requests | Throttled — check Retry-After header |

## Graph error response structure

```json
{
    "error": {
        "code": "Request_ResourceNotFound",
        "message": "Resource 'neexistuje' does not exist.",
        "innerError": {
            "date": "2026-06-17T...",
            "request-id": "...",
            "client-request-id": "..."
        }
    }
}
```

- `error.code` — machine-readable error identifier
- `error.message` — human-readable description
- `innerError.request-id` — useful for Microsoft support escalation

## PowerShell error handling

### Basic try/catch

```powershell
try {
    Get-MgUser -UserId "neexistuje@REDACTED.onmicrosoft.com" -ErrorAction Stop
} catch {
    $statusCode = $_.Exception.Response.StatusCode
    $message = $_.Exception.Message
    Write-Host "Error $statusCode : $message"
}
```

- `-ErrorAction Stop` is required — without it, Graph SDK errors are non-terminating and try/catch won't trigger

### Structured error handler

```powershell
function Invoke-SafeGraphQuery {
    param([string]$UserId)
    try {
        $user = Get-MgUser -UserId $UserId -Property DisplayName,Department -ErrorAction Stop
        Write-Host "Found: $($user.DisplayName)" -ForegroundColor Green
    } catch {
        $statusCode = $_.Exception.Response.StatusCode
        switch ($statusCode) {
            "NotFound"    { Write-Host "User '$UserId' not found." -ForegroundColor Yellow }
            "Forbidden"   { Write-Host "Insufficient permissions." -ForegroundColor Red }
            default       { Write-Host "Error $statusCode : $($_.Exception.Message)" -ForegroundColor Red }
        }
    }
}
```

## Debugging with -Debug

```powershell
Get-MgUser -UserId "user@REDACTED.onmicrosoft.com" -Debug
```

- Shows the raw HTTP request URL, method, and headers
- Shows the raw HTTP response status code and body
- Useful for verifying what the SDK is actually sending to Graph

## What I did

- Triggered a 404 error by querying a non-existent user.
- Triggered a 403 error by connecting with insufficient scopes (User.Read instead of User.Read.All).
- Triggered a 400 error with an invalid OData filter.
- Examined the error response structure in Graph Explorer.
- Built a structured error handler function with switch-based status code handling.
- Used the -Debug flag to inspect raw HTTP request/response details.

## Result

Common Graph API error scenarios were reproduced and handled. A reusable error handling function was built for production-style scripting.

## Lessons learned

- `-ErrorAction Stop` is mandatory for try/catch to work with Graph SDK cmdlets — without it, errors are non-terminating and silently continue.
- 403 (Forbidden) and 401 (Unauthorised) look similar but mean different things: 401 = no valid token, 403 = valid token but missing permissions.
- Graph error responses include `request-id` in `innerError` — this is essential when escalating to Microsoft support.
- The `-Debug` flag reveals the actual REST call the SDK makes — invaluable when the cmdlet documentation is unclear about which endpoint or parameters are used.
- Always check `error.code` (machine-readable) rather than `error.message` (can change) when building error handling logic.

## Evidence

```text
evidence/task-07-error-handling-and-debugging/
```
