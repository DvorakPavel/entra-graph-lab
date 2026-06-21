# Task 05 – Group Management

**Date:**
2026-06-20

## Goal

Manage groups through Microsoft Graph PowerShell SDK — create assigned and dynamic security groups, manage members and owners, and validate dynamic membership rules.

## Connection

```powershell
Connect-MgGraph -Scopes "Group.ReadWrite.All","User.Read.All"
```

## Group types

| Type | SecurityEnabled | MailEnabled | GroupTypes | Membership |
|---|---|---|---|---|
| Assigned security group | true | false | [] | Manual — add/remove members explicitly |
| Dynamic security group | true | false | ["DynamicMembership"] | Automatic — rule-based membership |
| Microsoft 365 group | true | true | ["Unified"] | Manual or dynamic |

## Assigned group operations

| Operation | Cmdlet |
|---|---|
| Create group | `New-MgGroup` |
| Add member | `New-MgGroupMember -DirectoryObjectId` |
| Remove member | `Remove-MgGroupMemberByRef -DirectoryObjectId` |
| List members | `Get-MgGroupMember` |
| Add owner | `New-MgGroupOwner -DirectoryObjectId` |
| List owners | `Get-MgGroupOwner` |
| Delete group | `Remove-MgGroup` |

## Dynamic group

```powershell
New-MgGroup -DisplayName "GRP-GRAPH-TEST-DYNAMIC-IT" `
    -MailEnabled:$false `
    -MailNickname "grp-graph-test-dynamic-it" `
    -SecurityEnabled:$true `
    -GroupTypes @("DynamicMembership") `
    -MembershipRule "(user.department -eq ""IT"")" `
    -MembershipRuleProcessingState "On"
```

- Membership is evaluated automatically based on the rule
- Processing can take 1–2 minutes after creation or rule change
- Members cannot be manually added or removed — the rule controls everything

## Member vs Owner

- **Member**: belongs to the group, gets access to resources assigned to the group
- **Owner**: can manage the group — add/remove members, change properties, and delete the group. Does not automatically become a member.

## What I did

- Listed existing groups in the tenant.
- Created an assigned security group and added two lab users as members.
- Added lab admin as group owner.
- Removed a member from the assigned group.
- Created a dynamic security group with a membership rule filtering by department.
- Verified that dynamic membership was automatically populated.
- Cleaned up both test groups.
- Disconnected the session.

## Result

Both assigned and dynamic security groups were created and managed entirely through the Graph PowerShell SDK. Dynamic membership rules were validated against existing user properties.

## Lessons learned

- Dynamic groups require `GroupTypes @("DynamicMembership")` and `MembershipRuleProcessingState "On"` — without both, the rule is ignored.
- You cannot manually add or remove members from a dynamic group — attempting it returns an error. Membership is exclusively controlled by the rules.
- `New-MgGroupMember` adds a member, but `Remove-MgGroupMemberByRef` removes one — the cmdlet names are asymmetric.
- Dynamic membership processing is not instant — allow 1–2 minutes for evaluation after group creation or rule changes.
- Group owners are not automatically members. A user can own a group without being part of it.
- `MailNickname` is required even for security groups that have no mail functionality.

## Evidence

```text
evidence/task-05-group-management/
```
