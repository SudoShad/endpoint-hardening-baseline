# Baseline pillars — rationale

This baseline is CIS-inspired and sized for **Desktop Support / IT Support / Jr Sysadmin** interviews. It is not a full CIS benchmark dump. Each pillar is something you can explain, deploy in a trial Intune tenant, and verify on a Windows lab VM.

Settings names below track current Microsoft Intune / Windows LAPS / BitLocker documentation (Endpoint security Disk encryption, Windows LAPS CSP, Defender Antivirus profiles). Portal labels move occasionally — treat click-paths in [`RUNBOOK.md`](RUNBOOK.md) as the operator source of truth.

---

## 1. Microsoft Defender Antivirus

**Why it matters on the helpdesk:** Most malware tickets start with “something feels wrong.” Real-time protection + cloud-delivered protection catch a large share of commodity threats before they become deskside rebuilds. Potentially Unwanted App (PUA) protection reduces toolbars and adware that generate repeat tickets.

**Baseline intent**

| Control | Intent |
|---------|--------|
| Real-time protection | On — scan on access |
| Cloud-delivered protection | On — MAPS / cloud block at good/high level |
| PUA protection | On (Audit → Block after pilot if noise is low) |
| Behavior monitoring | On |
| Network protection (optional stretch) | Audit first in lab |

**Helpdesk note:** If a legit line-of-business app is blocked, collect the detection name / hash, open an exclusion request through change control — do not teach users to “turn off Defender.”

---

## 2. Firewall (domain / private / public)

**Why it matters:** Enabled firewall profiles cut drive-by inbound exposure on laptops that leave the office. Domain vs private vs public profile awareness is a classic Jr Sysadmin interview topic.

**Baseline intent**

| Profile | Firewall | Inbound (high level) |
|---------|----------|----------------------|
| Domain | On | Block unsolicited; allow established |
| Private | On | Same; prefer known networks only |
| Public | On | Strict — block inbound that isn’t explicitly allowed |

**Helpdesk note:** App “can’t connect” tickets are often outbound application rules or third-party security suites fighting Windows Firewall — check `Get-NetFirewallProfile` and recent policy sync before disabling the firewall.

---

## 3. BitLocker (OS drive + Entra recovery)

**Why it matters:** Lost/stolen laptop tickets become breach tickets without encryption. Escrowing the recovery password to **Microsoft Entra ID** is what lets helpdesk pull a key without a sticky note under the keyboard.

**Baseline intent**

| Setting theme | Intent |
|---------------|--------|
| Require device encryption | Enabled for lab Windows endpoints with TPM |
| OS drive encryption | On (TPM protector; avoid requiring startup PIN for silent lab enablement) |
| Recovery password | Required; store / backup to Entra before enabling where policy allows |
| Silent encryption (lab option) | Only after confirming TPM + UEFI + Secure Boot + Entra join prerequisites |

**Helpdesk note:** Use Intune Devices → device → Recovery keys, or Graph tooling such as [helpdesk-graph-toolkit](https://github.com/SudoShad/helpdesk-graph-toolkit) `Get-BitLockerRecoveryKey.ps1`, once keys are in Entra. Never commit recovery keys to git.

---

## 4. Windows LAPS (via Entra / Intune)

**Why it matters:** Shared local-admin passwords are how one compromised workstation becomes every workstation. Windows LAPS rotates the managed local administrator password and backs it up to Entra so helpdesk can retrieve it per device.

**Baseline intent** (Windows LAPS CSP / Intune Account protection)

| Setting | Lab-friendly value |
|---------|-------------------|
| BackupDirectory | Back up to Microsoft Entra ID (`1`) |
| PasswordAgeDays | 30 (Entra minimum is 7) |
| PasswordComplexity | Large letters + small letters + numbers + special characters (`4`) |
| PasswordLength | 14+ |
| PostAuthenticationActions | Reset password and sign out (`3`) after grace period |
| AdministratorAccountName | Leave unset to manage built-in Administrator unless you maintain a custom account |

**Helpdesk note:** Retrieve LAPS passwords from Entra / Intune device blade — do not store them in ticket comments long-term.

---

## 5. Windows Update rings (conservative, helpdesk-friendly)

**Why it matters:** Zero deferral means feature updates surprise the floor overnight. Extreme deferral means you miss security fixes. Helpdesk-friendly rings buy a short validation window without looking careless.

**Baseline intent**

| Channel / deferral | Lab suggestion |
|--------------------|----------------|
| Servicing channel | General Availability (not Insider) |
| Quality update deferral | 3–7 days |
| Feature update deferral | 30–60 days (pick one and document it) |
| Feature update uninstall window | Keep default / allow rollback period where available |
| Deadline / grace | Gentle deadlines so laptops actually patch |

**Helpdesk note:** Pilot ring = IT + willing early adopters. Broad ring follows after a quiet week.

---

## 6. Local admin restriction / UAC (high level)

**Why it matters:** Daily-driver local admin turns every phishing click into a SYSTEM-level incident. UAC is not a silver bullet, but consent prompts create friction attackers dislike and give users a moment to rethink.

**Baseline intent (documentation-level — not a full AppLocker/WDAC build)**

- Prefer **standard users** for day-to-day work; elevate via LAPS or privileged access workstations for admin tasks.
- Keep UAC at the Windows default Notify level or higher — do not set “Never notify” via policy.
- Document who may be in local Administrators (break-glass / LAPS-managed account only on lab endpoints).

Full AppLocker / WDAC is intentionally out of scope for this baseline; call it a follow-on hardening track.

---

## Mapping to interview language

| They ask… | You point at… |
|-----------|---------------|
| “How do you harden Windows laptops?” | This baseline’s six pillars |
| “Where do recovery keys live?” | BitLocker → Entra escrow + helpdesk Graph toolkit |
| “How do you stop shared local admin?” | Windows LAPS + Entra backup |
| “How do you avoid update chaos?” | Update rings with deferral + pilot |
| “Have you done this in Intune?” | [`RUNBOOK.md`](RUNBOOK.md) click-paths + lab VERIFY commands |
