# Runbook — deploy each control in Intune

Operator-oriented click-paths for a **lab / trial** Microsoft Intune tenant. Portal UI labels drift; if a blade moved, search the setting name from the tables in [`../policies/`](../policies/).

**Prerequisites**

- Intune admin (Endpoint Security Administrator or equivalent custom RBAC)
- At least one Entra-joined or hybrid-joined Windows 10/11 lab device
- A **pilot** Azure AD / Entra security group with only lab devices
- Change note: what you are enabling, who is in pilot, rollback plan

**General pattern for every policy**

1. Create the profile in the correct Intune blade (below).
2. Assign to the pilot group only.
3. On the device: Settings → Accounts → Access work or school → Info → Sync, or wait for the next check-in.
4. Verify with [`VERIFY.md`](VERIFY.md).
5. Only then widen assignment.

---

## 1. Microsoft Defender Antivirus

**Preferred path — Endpoint security**

1. Sign in to [Microsoft Intune admin center](https://intune.microsoft.com).
2. Go to **Endpoint security** → **Antivirus** → **Create Policy**.
3. Platform: **Windows**. Profile: **Microsoft Defender Antivirus** (or **Windows Security experience** only if you need UI hardening separately).
4. Configuration settings — set at minimum:
   - Allow Realtime Monitoring → **Allowed**
   - Allow Cloud Protection → **Allowed**
   - Cloud Block Level → **High** (lab) or **High Plus** after pilot
   - PUA Protection → **PUA Protection on** (start with Audit in noisy labs if needed)
   - Allow Behavior Monitoring → **Allowed**
5. Assignments → pilot device group → **Create**.

**Alternate — Settings Catalog**

- Devices → Configuration → Create → Settings catalog → search **Microsoft Defender Antivirus** / **Windows Defender** and mirror the same settings.
- See [`../policies/defender.json`](../policies/defender.json) for a template-shaped reference (not a live export).

---

## 2. Firewall

**Preferred path — Endpoint security**

1. **Endpoint security** → **Firewall** → **Create Policy**.
2. Platform: **Windows**. Profile: **Windows Firewall**.
3. Enable firewall for **Domain**, **Private**, and **Public** profiles.
4. Keep inbound defaults restrictive on Public; avoid “allow all inbound” shortcuts.
5. Assign to pilot → Create.

**Documentation table:** [`../policies/firewall.md`](../policies/firewall.md)

---

## 3. BitLocker (OS drive + Entra recovery)

**Preferred path — Endpoint security Disk encryption**

1. **Endpoint security** → **Disk encryption** → **Create Policy**.
2. Platform: **Windows**. Profile: **BitLocker**.
3. Configure (lab silent-friendly starting point — adjust for your hardware):
   - **Require Device Encryption** → Enabled
   - Recovery password required; **save BitLocker recovery information to Microsoft Entra ID**
   - Prefer storing recovery info in Entra **before** enabling BitLocker when the setting is available
   - For silent lab enablement: **Allow Warning For Other Disk Encryption** → Disabled only after you confirm no third-party encryption; **Allow Standard User Encryption** → Enabled if users are not local admins
   - TPM startup PIN/key → **Do not allow** if you want silent encryption (PIN requires user interaction)
4. Assign to pilot devices that meet prerequisites (TPM 1.2+, UEFI, Secure Boot, WinRE, Entra join).
5. Monitor: **Devices** → **Monitor** → **Encryption report**, and per-device **Recovery keys**.

**Do not** use silent encryption on machines with unknown third-party disk encryption.

**Documentation:** [`../policies/bitlocker.md`](../policies/bitlocker.md)

---

## 4. Windows LAPS

**Preferred path — Endpoint security Account protection**

1. Confirm Windows LAPS prerequisites (supported Windows builds; Entra joined or hybrid joined for Entra backup).
2. **Endpoint security** → **Account protection** → **Create Policy**.
3. Platform: **Windows**. Profile: **Local admin password solution (LAPS)** / Windows LAPS.
4. Set (aligned with Windows LAPS CSP):
   - **Backup Directory** → Back up the password to Microsoft Entra only
   - **Password Age Days** → 30 (minimum 7 when backing up to Entra)
   - **Password Complexity** → Large letters + small letters + numbers + special characters
   - **Password Length** → 14 (or higher)
   - **Post Authentication Actions** → Reset password and sign out (after grace period)
5. Assign to pilot → Create.
6. Retrieve test password from the device object in Entra / Intune after the first rotation to prove escrow works.

**Documentation:** [`../policies/laps.md`](../policies/laps.md)

---

## 5. Windows Update rings

**Path — Devices / Windows updates**

1. **Devices** → **Windows updates** → **Update rings** (or **Quality / Feature updates** blades if your tenant shows the newer update experience).
2. **Create profile** → Windows 10 and later.
3. Suggested lab values (document whatever you pick):
   - Microsoft product updates → Allow
   - Servicing channel → **General Availability**
   - Quality update deferral period (days) → **7**
   - Feature update deferral period (days) → **30** (or 60 if you want a wider cushion)
   - Automatic update behavior → Install and restart at maintenance / deadline settings you can support
4. Assign to pilot first; create a second broader ring later.
5. Exclude Insider channels from production-like lab rings.

**Documentation:** [`../policies/update-rings.md`](../policies/update-rings.md)

---

## 6. Local admin / UAC (high level)

There is no single “turn on CIS” button. Practical lab steps:

1. Ensure day-to-day lab users are **standard users** (not permanent local Administrators).
2. Rely on **Windows LAPS** for the managed local admin account instead of a shared password.
3. Optional Settings Catalog: search **User Account Control** and keep behavior at Notify / default or stricter — never “Never notify.”
4. Document in your ticket / change log who retains local admin on the pilot VM.

---

## Importing JSON samples

Files under [`../policies/`](../policies/) are **templates for recreation**:

- Prefer rebuilding via the UI using the tables (most reliable across portal changes).
- If you use Graph / import tooling, strip any tenant-specific IDs — these samples intentionally contain **none**.
- Do not paste samples into production without review.

---

## Rollback (lab)

| Control | Rollback idea |
|---------|----------------|
| Defender / Firewall / LAPS / Update ring | Remove assignment or delete profile; sync device |
| BitLocker | Suspend with `manage-bde -protectors -disable C:` only in lab with recovery key in hand; decrypt is slow — prefer suspend for troubleshooting |
| Bad update ring | Move device to a ring with zero deferral / pause updates temporarily |

Always confirm recovery key / LAPS password visibility in Entra **before** stressing encryption or admin rotation in lab.
