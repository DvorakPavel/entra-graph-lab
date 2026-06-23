# Task 10 – Directory Roles & PIM via Graph

**Date:**
2026-06-23

## Goal

Manage directory role assignments through Microsoft Graph PowerShell SDK — list role definitions, assign and remove users from roles, and explore Privileged Identity Management (PIM) eligible assignments.

## Connection

```powershell
Connect-MgGraph -Scopes "RoleManagement.ReadWrite.Directory","Directory.Read.All"
```

## Key concepts

### Role Template vs Directory Role

| Concept | Description |
|---|---|
| Role Template | Blueprint of a role — all ~90 built-in roles exist as templates regardless of usage |
| Directory Role | An activated role — only appears after at least one user has been assigned to it |

A role template must be "activated" (instantiated as a Directory Role) before members can be added. This happens automatically through the portal but must be done explicitly via API.

### Permanent vs Eligible (PIM)

| Type | Description | Security |
|---|---|---|
| Permanent (active) | User has the role 24/7 | Higher risk — always-on privileges |
| Eligible (PIM) | User can activate the role on demand, with time limit and optional approval | Lower risk — just-in-time access |

PIM requires Entra ID P2 license.

## Operations

| Operation | Cmdlet |
|---|---|
| List role templates | `Get-MgDirectoryRoleTemplate` |
| List active roles | `Get-MgDirectoryRole` |
| Get role members | `Get-MgDirectoryRoleMember -DirectoryRoleId` |
| Activate a role | `New-MgDirectoryRole -RoleTemplateId` |
| Add member | `New-MgDirectoryRoleMemberByRef -DirectoryRoleId -BodyParameter @{"@odata.id" = "..."}` |
| Remove member | `Remove-MgDirectoryRoleMemberByRef -DirectoryRoleId -DirectoryObjectId` |
| PIM eligible | `Get-MgRoleManagementDirectoryRoleEligibilityScheduleInstance` |
| PIM active | `Get-MgRoleManagementDirectoryRoleAssignmentScheduleInstance` |

## What I did

- Listed all built-in directory role templates (~90 roles).
- Listed currently activated roles (roles with assigned members).
- Retrieved members of the Global Administrator role.
- Assigned a lab user to the Helpdesk Administrator role via Graph API.
- Verified the assignment and then removed the user from the role.
- Queried PIM eligible assignments — found an eligible User Administrator assignment with time-limited scope (April 2026 – April 2027).
- Compared permanent (active) vs eligible (PIM) assignment counts.
- Disconnected the session.

## Result

The directory role lifecycle was managed through the Graph PowerShell SDK — listing, assigning, and removing role memberships. PIM eligible assignments were successfully queried and compared with permanent assignments.

## Lessons learned

- `New-MgDirectoryRoleMember` does not exist in Graph SDK v2 — use `New-MgDirectoryRoleMemberByRef` with a `@odata.id` body parameter pointing to the directory object URL.
- Role templates (~90 built-in) always exist, but Directory Roles only appear after activation — you must activate a role before assigning members via API.
- `Get-MgDirectoryRoleMember` returns directory objects, not user objects — you need an additional `Get-MgUser` call to get display names and UPNs.
- PIM eligible assignments can reference service principals or groups, not just users — `Get-MgUser` on a PrincipalId returns 404 if it's not a user. Use `Get-MgDirectoryObject` for safe resolution.
- `Get-MgRoleManagementDirectoryRoleEligibilityScheduleInstance` returns raw GUIDs for PrincipalId and RoleDefinitionId — resolve them with `Get-MgUser`/`Get-MgDirectoryObject` and `Get-MgRoleManagementDirectoryRoleDefinition` for human-readable output.
- Removing the last Global Administrator is blocked by Entra ID — there must always be at least one permanent Global Admin.
- In production, all admin roles should use PIM eligible assignments instead of permanent — this is a core SC-300 best practice (least privilege + just-in-time access).
- The `RoleManagement.ReadWrite.Directory` scope is powerful — it allows both reading and modifying role assignments. Use `RoleManagement.Read.Directory` for read-only scripts.
- Always write explicit commands in scripts — avoid relying on variables set in earlier steps. Anyone reading the script (including your future self) should understand each command independently.

## Evidence

```text
evidence/task-10-directory-roles-pim/
```
