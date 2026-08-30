# Delegation Permission Testing

With the revised OU structure, security groups, and delegated permissions in place, I moved on to validating the RBAC model from the perspective of the individual IT roles.

The purpose of this stage is to test both sides of the permission model:

- Intended operations were allowed.
- Higher-risk operations were denied.
- Permissions applied only within the intended scope.
- Troubleshooting and diagnostic capabilities matched each role.

This provides practical evidence that the delegated permissions and tiered administration model are functioning as intended.

---

## Current State - WIP

At this stage:

## Roles Tested

| Role | Security group | Testing page | Result |
| --- | --- | --- | --- |
| Help Desk | `EL_HelpDesk` | [Help Desk Testing](delegated-testing/helpdesk_testing.md) | Pass |
| SysAdmin | `EL_SysAdmins` | [SysAdmin Testing](delegated-testing/sysadmin_testing.md) | Pass |
| Administrative SysAdmin | `EL_Adm_SysAdmins` | [Administrative SysAdmin Testing](delegated-testing/administrative_sysadmin_testing.md) | Pending |


!!! note "Testing objective"
    The configuration itself has already been documented in the [Active Directory Delegation & RBAC](ad_delegation_rbac.md) section. This section focuses on validating that those permissions behave as intended in practice.

---

## Testing Order

The roles are being tested in the following order:

1. Help Desk — `EL_HelpDesk`
2. SysAdmin standard tier — `EL_SysAdmins`
3. SysAdmin administrative tier — `EL_Adm_SysAdmins`
4. NetAdmin standard tier — `EL_NetAdmin`
5. Network administrative tier - `EL_Adm_NetAdmin`
6. SecAdmin standard tier — `EL_SecAdmin`
7. SecAdmin administrative tier - `EL_Adm_SecAdmin`
8. IT Management — `EL_ITManagement`
9. IT Management - `EL_Adm_ITManagement`

Testing starts with the narrowest role before moving into accounts with broader administrative permissions.

---


