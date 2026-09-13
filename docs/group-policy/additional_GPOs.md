## Windows Update Deferral GPO

After getting prompts to update my Windows VMs, I wanted to create a GPO that would defer updates to when I wanted to update the VMs.

!!! note "Real-world justification"
    In real world practice this would be utilised for risk mitigation in regards to possible bugs when updating, and would allow for greater control of when updates should occur.

To create the GPO, I ran:`Win + R > gpmc.msc`

I then created a new GPO and proceeded to edit it:
`Computer Configuration → Policies → Administrative Templates → Windows Components →
Windows Update → Manage updates offered from Windows Update`

I then edited:

- **Select when Quality Updates are received** — 14 days
- **Select when Preview Builds and Feature Updates are received** — 14 days

This was then linked to the Workstations OU.

!!! note "Placeholder — WSUS"
    This GPO-based deferral is a placeholder until WSUS is properly configured in Stage 4. Once WSUS is in place, update approval should flow through WSUS rather than GPO deferral alone, giving full control over which updates are approved and distributed, not just when clients are allowed to check for them.

I then ran `gpupdate /force` on the clients and restarted, confirming the policy applied via:

```powershell
gpresult /r /scope:computer
```

The GPO showed correctly under Applied Group Policy Objects.

---

### Final Validation Summary

| Test                                                    | Expected result | Actual result                                      | Status |
| -------------------------------------------------------- | ---------------- | ---------------------------------------------------- | ------ |
| GPO created and linked to Workstations OU               | Applied           | GPO appeared correctly under Applied GPOs             | Pass   |
| Quality Updates deferred 14 days                        | Configured        | Setting applied as configured                         | Pass   |
| Preview Builds and Feature Updates deferred 14 days      | Configured        | Setting applied as configured                         | Pass   |
| GPO confirmed applied on client after refresh and restart | Applied          | Confirmed via `gpresult /r /scope:computer`           | Pass   |

---

### Key Takeaways

- Deferring updates via GPO is a practical, low-effort way to gain control over patch timing without needing a dedicated update management server, useful as an interim measure before WSUS exists.
- This is explicitly a placeholder, not the intended long-term solution. GPO deferral only controls *when* a client checks for updates, not *which* updates are approved for the environment; WSUS (Stage 4) provides that finer-grained approval control and should replace this once built.
- As with other Computer Configuration policies, confirming the change with `gpresult /r /scope:computer` after a restart, rather than assuming success from the GPO editor alone, remains the reliable way to validate the policy actually applied.
