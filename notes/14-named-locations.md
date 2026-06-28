# Task 14 – Named Locations & Network Conditions

**Date:**
2026-06-28

## Goal

Create and manage named locations through Microsoft Graph PowerShell SDK — IP-based trusted locations and country-based locations used as conditions in Conditional Access policies.

## Connection

```powershell
Connect-MgGraph -Scopes "Policy.ReadWrite.ConditionalAccess","Policy.Read.All"
```

## Key concepts

### Named location types

| Type | OData type | Use case |
|---|---|---|
| IP-based | `#microsoft.graph.ipNamedLocation` | Define trusted office networks, VPN ranges, known IP addresses |
| Country-based | `#microsoft.graph.countryNamedLocation` | Block or allow sign-ins from specific countries/regions |

### IsTrusted flag

- Only available on IP-based named locations
- When `IsTrusted = $true`, CA policies can use the condition "from trusted location"
- Common pattern: skip MFA when signing in from a trusted office network
- SC-300 best practice: do NOT skip MFA even from trusted locations (zero trust), but the capability is important to understand

### How named locations connect to CA policies

Named locations are referenced in CA policy conditions:

```json
"conditions": {
    "locations": {
        "includeLocations": ["All"],
        "excludeLocations": ["trusted-location-id"]
    }
}
```

- `includeLocations: ["All"]` + `excludeLocations: [trusted-id]` = apply policy everywhere except trusted locations
- `includeLocations: [country-id]` = apply policy only for sign-ins from specific countries

### Country codes (ISO 3166-1 alpha-2)

| Code | Country | Common reason to block |
|---|---|---|
| RU | Russia | High volume of credential attacks |
| CN | China | State-sponsored threat actors |
| KP | North Korea | Sanctions and threat activity |

## Operations

| Operation | Cmdlet |
|---|---|
| List all | `Get-MgIdentityConditionalAccessNamedLocation` |
| Get by ID | `Get-MgIdentityConditionalAccessNamedLocation -NamedLocationId` |
| Create | `New-MgIdentityConditionalAccessNamedLocation -BodyParameter` |
| Update | `Update-MgIdentityConditionalAccessNamedLocation -NamedLocationId -BodyParameter` |
| Delete | `Remove-MgIdentityConditionalAccessNamedLocation -NamedLocationId` |

## What I did

- Listed all existing named locations in the tenant.
- Created an IP-based trusted named location with a CIDR range.
- Created a country-based named location blocking RU, CN, KP.
- Updated the IP-based location to add a second CIDR range.
- Retrieved the full JSON detail of a named location.
- Generated an overview table of all locations with type and trust status.
- Cleaned up all test locations.
- Disconnected the session.

## Result

Both IP-based and country-based named locations were created, updated, and deleted through the Graph PowerShell SDK. The relationship between named locations and Conditional Access policy conditions was documented.

## Lessons learned

- Named locations require `@odata.type` in the body parameter — without it, the API doesn't know whether to create an IP-based or country-based location. This is different from most other Graph cmdlets.
- IP ranges must use CIDR notation (e.g. `203.0.113.0/24`) and each range needs its own `@odata.type` (`#microsoft.graph.iPv4CidrRange` or `#microsoft.graph.iPv6CidrRange`).
- `IsTrusted` is only available on IP-based locations — country-based locations cannot be marked as trusted.
- Updating IP ranges replaces the entire array — you must include all existing ranges plus the new ones, otherwise the old ranges are removed.
- Country codes follow ISO 3166-1 alpha-2 — use `"US"` not `"USA"` or `"United States"`.
- Named locations are not policies themselves — they are reusable conditions that CA policies reference by ID. Deleting a named location that is referenced by a CA policy causes that condition to become invalid.
- `IncludeUnknownCountriesAndRegions` controls whether sign-ins from unresolved IP addresses (no geo-location match) are included — setting this to `$true` blocks users behind VPNs or anonymisers that mask their country.

## Evidence

```text
evidence/task-14-named-locations/
```
