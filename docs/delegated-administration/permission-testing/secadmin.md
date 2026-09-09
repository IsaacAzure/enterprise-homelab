## Testing — SecAdmin and Admin Tier SecAdmin

I wanted SecAdmins to be able to read all user information, run RSoP Logging and Planning, and read event logs.

When delegating the permissions, I added both the standard and admin tier to the same flow to add permissions concurrently.

I ran:

```powershell
Add-ADGroupMember -Identity "Event Log Readers" -Members "EL_SecAdmin", "EL_Adm_SecAdmins"
```

to give both tiers access to view event logs.

I validated via:

```powershell
Get-ADGroupMember -Identity "Event Log Readers"
```

which showed the two groups now as Event Log Readers members.

![Event log reader members](../../assets/images/event_log_members.png)

I then created a shared directory named `Security-Docs` and gave the standard tier **Change** share permissions.

In the Security tab, I added `EL_SecAdmin` and gave them **Modify** permissions, allowing them to write to the directory.

I then created a test file named "SecAdmin created this" as `EL_SecAdmin`, and this was successful.

![SecAdmin test file](../../assets/images/secadmin_test_file.png)

I tested with `EL_Adm_SecAdmins` and verified that this group could not create files in the directory, as per my design choice: the admin tier is scoped to state-changing actions such as deployment and patch scheduling, not documentation writing, so it deliberately has no write access to this share.

!!! note "I fixed the naming consistency between SecAdmin and Admin SecAdmins"
	Going forward `EL_SecAdmin` has been updated fully to be `EL_SecAdmins`. So this will be reflected in any notes and screenshots after this point.

!!! success "SecAdmin standard/admin tier write access confirmed correctly scoped"
    `EL_SecAdmin` can write to the documentation share as intended. `EL_Adm_SecAdmins` correctly cannot, consistent with the role's design: standard tier covers observational and documentation work, admin tier covers state-changing actions only.
