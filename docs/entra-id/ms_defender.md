# MS Defender

## Setting Up Microsoft Defender for Business

**Admin Center → Security → "Welcome to MS Defender for Business"** (this may require going into settings first to prompt the system to load and connect) **→ Get started**:

**User access**

* Myself (my Global Admin account) — Security Admin
* Rache Batmoss-Admin — Security Reader (would be Admin in a real deployment)

**Email notifications**

* Myself — Incidents and Vulnerabilities
* Rache Admin

**Add Windows devices**

* Scope set to **All devices** — any newly added devices will be automatically onboarded

**Security settings**

* Kept **Use Intune** selected, rather than Defender's own device management

## Connecting Defender to Intune

Back in **Intune admin center → Endpoint security → Microsoft Defender for Endpoint** (under **Setup** in the left nav):

* **Connection status** — Enabled
* **Windows devices connected to endpoint** setting — On

The **Overview** page, however, showed devices as **not onboarded** despite the connector itself reporting as enabled — the connector being on only means Intune and Defender can talk to each other, it doesn't onboard any devices by itself.

## Onboarding Devices to Defender

To actually onboard a device, I needed a dedicated EDR policy rather than relying on the connector alone:

1. **Intune admin center → Endpoint security → Endpoint detection and response → Create Policy.** Set Platform to **Windows 10, 11 and later** and Profile to **Endpoint detection and response**.
2. Set **Microsoft Defender for Endpoint client configuration package type** to **Auto from connector** (recommended) — this provisions the device automatically through MDM using the connector already enabled above, rather than a manual onboarding script.
3. On the Assignments tab, targeted the same group already used for the compliance policy — `Intune Workstations` (the dynamic device group) — so WS_01, and anything matching the `WS_*` naming pattern, gets onboarded automatically with no extra group management.
4. On WS_01: **Settings → Accounts → Access work or school → (Entra domain) → Info → Sync**, to force the device to check in and pull the new policy immediately rather than waiting for the next scheduled MDM sync interval.
5. Confirmed onboarding on the device itself — the **Sense** service (Windows Defender Advanced Threat Protection Service) running in Task Manager → Services, and the registry key `HKLM\SOFTWARE\Microsoft\Windows Advanced Threat Protection\Status` showing `OnboardingState = 1`.
6. Confirmed in the portals — **security.microsoft.com → Assets → Devices** showed WS_01 as onboarded.

!!! note "WS_02 onboarded automatically, no extra steps needed"
    I quickly joined WS_02 to the domain around the same time. Since it also matches the `WS_*` naming pattern, it picked up the `Intune Workstations` dynamic group membership and the EDR onboarding policy the same way WS_01 did — no separate onboarding steps were needed for it.

## A Reporting Lag Between Portals

Even with both devices confirmed onboarded in Defender's own portal, Intune's **Endpoint security overview** widget still showed **1 not onboarded**, even after refreshing.

!!! note "Two different data pipelines, not a fault"
    Defender's Assets → Devices list reflects the device's own onboarding heartbeat directly (the Sense service reporting in) — the fastest signal available. Intune's overview widget, on the other hand, aggregates via the Intune-Defender connector sync, which runs on its own separate schedule and can lag behind by anywhere from minutes to a couple of hours. Refreshing the overview page only re-queries Intune's last cached sync from Defender — it doesn't force a new sync on demand. Since both **All devices** and Defender's own portal already showed both machines correctly, the underlying state was right; the summary widget just hadn't caught up yet.

---

## Final Validation Summary

| Test | Result |
|---|---|
| Defender for Business set up (users, notifications, device scope) | ✅ Done |
| Security settings kept on Intune for device management | ✅ Confirmed |
| Intune–Defender connector — Connection status | ✅ Enabled |
| Intune–Defender connector — Windows devices connected setting | ✅ On |
| Devices onboarded via connector alone (no EDR policy) | ❌ Not onboarded — connector enables the link, not onboarding itself |
| EDR onboarding policy created and assigned to `Intune Workstations` | ✅ Done |
| WS_01 onboarded (Sense service running, registry OnboardingState = 1) | ✅ Confirmed |
| WS_01 shown as onboarded in security.microsoft.com | ✅ YES |
| WS_02 onboarded automatically via same dynamic group | ✅ YES, no extra steps |
| Intune Endpoint security overview widget reflecting both onboarded | ⏳ Lagging — expected, separate sync pipeline |

---

## Key Takeaways

* **Enabling the Intune–Defender connector does not onboard any devices by itself.** It only establishes that the two services can exchange data — actual onboarding still requires a dedicated Endpoint detection and response policy assigned to the devices.
* **"Auto from connector" reuses the connector you've already configured**, so there's no need for a manual onboarding script on devices that are already Intune-managed — consistent with how the rest of this lab is built around MDM-driven management rather than manual per-device setup.
* **Assigning the EDR policy to the same dynamic device group used elsewhere (`Intune Workstations`) means any future `WS_*` device onboards automatically** — exactly what happened with WS_02, which needed no separate steps once it joined the domain.
* **Defender's own portal and Intune's overview widget don't refresh in lockstep.** Defender's Assets → Devices list is closer to a live signal from the device itself; Intune's summary widget depends on a separate connector sync and can genuinely lag behind, even after a manual refresh.
