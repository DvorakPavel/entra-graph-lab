# Task 01 – Graph Explorer Fundamentals

**Date:**
2026-06-17

## Goal

Learn the basics of Microsoft Graph API using Graph Explorer — endpoints, HTTP methods, OData query parameters, and permission consent.

## Graph Explorer

- URL: https://developer.microsoft.com/en-us/graph/graph-explorer
- Signed in with: Lab admin account
- API versions available: v1.0 (production), beta (preview)

## Basic endpoints tested

| Endpoint | Description |
|---|---|
| `GET /v1.0/me` | Current signed-in user profile |
| `GET /v1.0/users` | All users in the tenant |
| `GET /v1.0/groups` | All groups in the tenant |

## OData query parameters

| Parameter | Example | Purpose |
|---|---|---|
| `$select` | `$select=displayName,userPrincipalName` | Return only specified properties |
| `$filter` | `$filter=department eq 'IT'` | Filter results by condition |
| `$top` | `$top=5` | Limit number of results |
| `$orderby` | `$orderby=displayName` | Sort results |
| `$expand` | `$expand=memberOf` | Expand related objects inline |

## Combined query example

```
GET https://graph.microsoft.com/v1.0/users?$filter=department eq 'HR'&$select=displayName,jobTitle&$orderby=displayName
```

## Permissions observed

- `/me` — requires: User.Read
- `/users` — requires: User.Read.All
- Consent was granted through Graph Explorer's permission consent dialog

## v1.0 vs beta

- v1.0: stable, production-ready, fewer properties
- beta: preview, more properties and endpoints, may change without notice

## What I did

- Signed into Graph Explorer with the lab admin account.
- Executed basic GET requests against /me, /users, and /groups endpoints.
- Tested each OData query parameter individually ($select, $filter, $top, $orderby, $count, $expand).
- Combined multiple OData parameters in a single query.
- Reviewed the permissions consent dialog and granted required permissions.
- Compared v1.0 and beta endpoint responses for the same query.

## Result

Graph Explorer was used to query the tenant directory, filter and shape results using OData parameters, and understand the permission consent model. The difference between v1.0 and beta endpoints was observed.

## Lessons learned

- OData query parameters go in the URL query string, not in the Request Body — GET requests ignore the body.
- `$count=true` requires the header `ConsistencyLevel: eventual`, otherwise it returns an error.
- Graph Explorer returns only default properties — use `$select` to get specific non-default properties.
- v1.0 is stable and guaranteed; beta has more features but can change without notice. Always use v1.0 for production scripts.
- `/me` uses delegated User.Read (current user only), `/users` requires User.Read.All (tenant-wide access) — different permission scope for what looks like a similar query.

## Evidence

```text
evidence/task-01-graph-explorer-fundamentals/
```
