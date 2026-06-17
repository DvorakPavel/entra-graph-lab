# Task 02 – Permission Model Deep Dive

**Date:**
2026-06-17

## Goal

Understand how Microsoft Graph permissions work — delegated vs application, consent types, least privilege, and the relationship between scopes, roles, and effective permissions.

## Delegated vs Application permissions

| Aspect | Delegated | Application |
|---|---|---|
| Runs as | Signed-in user + app | App only (no user) |
| Token claim | `scp` (scopes) | `roles` |
| Effective access | Intersection of app permissions AND user privileges | App permissions only |
| Consent | User or admin | Admin only |
| Use case | Interactive apps, Graph Explorer | Background services, daemons, automation scripts |

## Consent types

| Type | Who grants | When used |
|---|---|---|
| User consent | End user | Low-privilege delegated permissions (e.g. User.Read) |
| Admin consent | Global Admin or privileged role | High-privilege or application permissions |
| Admin consent workflow | End user requests, admin approves | When user consent is restricted |

## Least privilege examples

| Query | Minimum permission | Why |
|---|---|---|
| `GET /v1.0/me` | User.Read | Own profile only |
| `GET /v1.0/users` | User.Read.All | All users in tenant |
| `GET /v1.0/users?$select=displayName` | User.Read.All | $select limits data, not access scope |
| `PATCH /v1.0/users/{id}` | User.ReadWrite.All | Write operation requires Write permission |

## Access token inspection

- Tool: https://jwt.ms
- Delegated token contains `scp` claim with granted scopes
- Application token contains `roles` claim with granted roles
- Token also shows: `aud` (audience), `iss` (issuer), `exp` (expiration)

## Effective permissions

- Delegated: effective access = app permissions & user privileges
- Application: effective access = app permissions
- A delegated app with User.ReadWrite.All used by a non-admin user cannot modify other users — the user's own privileges limit it

## What I did

- Reviewed the Modify Permissions panel in Graph Explorer.
- Examined API permissions on the existing App Registration in Entra admin center.
- Reviewed User consent settings and Admin consent workflow configuration.
- Decoded an access token at jwt.ms and identified the `scp` claim.
- Tested the same Graph query with different permission levels to observe the difference.
- Verified that $select reduces response data but does not reduce required permissions.

## Result

The Microsoft Graph permission model was explored across delegated and application contexts. The relationship between consent types, least privilege, and effective permissions was validated through hands-on testing.

## Lessons learned

- Delegated permissions are bounded by the signed-in user's own privileges — the app can never do more than the user could.
- Application permissions bypass user context entirely, which makes them powerful but dangerous — always prefer delegated when a user is present.
- `$select` optimises response payload but does not reduce the permission scope required.
- Decoding the access token at jwt.ms is the fastest way to verify what permissions are actually granted vs what was requested.
- Admin consent is required for all application permissions and for high-privilege delegated permissions — there is no user self-service path for these.

## Evidence

```text
evidence/task-02-permission-model-deep-dive/
```
