# Task 06 – Pagination & Throttling

**Date:**
2026-06-21

## Goal

Understand how Microsoft Graph handles large result sets through pagination and how it protects itself from overload through throttling (HTTP 429).

## Pagination

### How it works

- Graph API returns results in pages (default page size varies by endpoint)
- If more results exist, the response includes `@odata.nextLink` — a URL to the next page
- To get all results, follow `@odata.nextLink` until it's no longer present in the response

### PowerShell SDK pagination

| Approach | Cmdlet parameter | Behavior |
|---|---|---|
| First page only | (default) | Returns up to 100 results |
| All pages | `-All` | Automatically follows @odata.nextLink until all results are retrieved |
| Custom page size | `-All -PageSize 50` | Retrieves all results in chunks of 50 |
| Limited results | `-Top 5` | Returns exactly 5 results (server-side limit) |

### -All warnings

`-All` on a large tenant (thousands of users, millions of sign-in logs) loads everything into memory. For production scripts, use `-Top` with manual pagination or stream processing instead.

## Throttling

### How it works

- Graph API enforces rate limits per app, per tenant
- When exceeded, the API returns **HTTP 429 (Too Many Requests)**
- The response includes a `Retry-After` header indicating how many seconds to wait
- The Graph PowerShell SDK automatically retries after the specified delay

### Rate limit headers

| Header | Purpose |
|---|---|
| `x-ms-resource-unit` | Cost of the current request |
| `RateLimit-Limit` | Maximum requests allowed in the window |
| `RateLimit-Remaining` | Remaining requests in the current window |
| `Retry-After` | Seconds to wait before retrying (only on 429) |

### SDK automatic retry

The Microsoft Graph PowerShell SDK handles 429 responses transparently — it reads the `Retry-After` header, waits, and retries the request without any manual intervention.

## $count and ConsistencyLevel

- `$count=true` requires the header `ConsistencyLevel: eventual`
- Without the header, the request returns an error
- `eventual` means the count may be slightly delayed (not real-time) but is accurate enough for most use cases
- In PowerShell: `-CountVariable varName -ConsistencyLevel eventual`

## What I did

- Observed @odata.nextLink in Graph Explorer by requesting a small page size.
- Compared results with and without the -All parameter in PowerShell.
- Used -PageSize to control pagination chunk size.
- Queried sign-in audit logs as a larger dataset example.
- Examined response headers in Graph Explorer — rate limit headers were not present on a small dev tenant (throttling does not trigger at this scale).
- Used $count with ConsistencyLevel to get total user count.

## Result

Pagination and throttling mechanisms were explored both in Graph Explorer (raw REST) and PowerShell SDK. The SDK's automatic handling of pagination (-All) and throttling (429 retry) was validated.

## Lessons learned

- Always check for `@odata.nextLink` in raw REST responses — without it, you only get the first page and silently miss data.
- `-All` is convenient but dangerous on large datasets — it loads everything into memory. For production, use `-Top` with streaming or export.
- `$count` requires `ConsistencyLevel: eventual` — this is a common gotcha that produces confusing errors if forgotten.
- The Graph SDK handles 429 automatically, but custom scripts using raw REST (Invoke-RestMethod) need manual retry logic.
- Rate limit headers (`x-ms-resource-unit`, `RateLimit-Limit`, `RateLimit-Remaining`) were not visible on a small dev tenant — throttling only triggers at scale.
- Different endpoints have different throttling limits — user queries are more generous than report/audit log queries.

## Evidence

```text
evidence/task-06-pagination-and-throttling/
```
