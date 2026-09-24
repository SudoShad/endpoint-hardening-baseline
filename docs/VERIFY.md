# Verify on a Windows endpoint

Run these on a **lab** Windows 10/11 VM after Intune sync. Prefer an elevated PowerShell session where noted. Commands are for when you have a lab VM later — this repo does not require a live device to be useful as documentation.

**Sync first**

```powershell
# Soft nudge — or use Company Portal / Settings > Access work or school > Sync
Start-Process "companyportal:"
```

Confirm the device shows the policies as **Succeeded** in Intune (Devices → select device → Device configuration / Device security).

---

## Defender Antivirus

```powershell
Get-MpComputerStatus |
  Select-Object AMServiceEnabled, AntispywareEnabled, RealTimeProtectionEnabled,
                IoavProtectionEnabled, AntispywareSignatureLastUpdated,
                AMRunningMode, IsVirtualMachine

Get-MpPreference |
  Select-Object DisableRealtimeMonitoring, MAPSReporting, PUAProtection,
                DisableBehaviorMonitoring, SubmitSamplesConsent
```

**Expect (baseline direction)**

- `RealTimeProtectionEnabled` → `True`
- `DisableRealtimeMonitoring` → `False`
- `PUAProtection` → non-zero / enabled (exact enum depends on build)
- Signatures recently updated

---

## Firewall profiles

```powershell
Get-NetFirewallProfile |
  Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction |
  Format-Table -AutoSize
```

**Expect:** Domain, Private, and Public → `Enabled = True`. Public inbound should not be a wide Allow.

---

## BitLocker (OS drive)

```powershell
manage-bde -status C:
# or
Get-BitLockerVolume -MountPoint 'C:' |
  Select-Object MountPoint, VolumeStatus, ProtectionStatus, EncryptionPercentage,
                KeyProtector
```

**Expect**

- Protection **On** after encryption completes
- A **RecoveryPassword** protector present
- In Intune / Entra: device → **Recovery keys** shows a Key ID (confirm escrow — do not paste keys into tickets long-term or into git)

---

## Windows LAPS

Module / cmdlets vary by Windows build. On current Windows 11 with LAPS components:

```powershell
# Policy / status oriented (names available on builds with Windows LAPS)
Get-Command *Laps* -ErrorAction SilentlyContinue

# Common checks when Microsoft.Windows.LAPS cmdlets are present:
Get-LapsPolicy -ErrorAction SilentlyContinue
Get-LapsAADPassword -ErrorAction SilentlyContinue
# Legacy / alternate:
# Get-LapsDiagnostics
```

**Expect**

- Policy shows BackupDirectory targeting Entra
- After age/rotation, password is retrievable from Entra / Intune for that device
- Event Viewer → Applications and Services Logs → Microsoft → Windows → LAPS → Operational for success/failure events

If cmdlets are missing, verify from Intune Account protection policy status and Entra device LAPS blade instead — still valid for lab writeups.

---

## Windows Update ring

```powershell
# Deferral / ring influence often surfaces via registry / MDM
reg query "HKLM\SOFTWARE\Microsoft\PolicyManager\current\device\Update" /s 2>$null
reg query "HKLM\SOFTWARE\Microsoft\WindowsUpdate\UX\Settings" 2>$null

# What's pending / last result
Get-WindowsUpdateLog -ErrorAction SilentlyContinue  # may redirect; optional
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 8
```

**Expect**

- Device appears in the correct Update ring assignment in Intune
- Feature/quality deferrals match what you configured (portal is authoritative if registry paths differ by build)

---

## Local admin / UAC (spot checks)

```powershell
# Who is local admin?
Get-LocalGroupMember -Group 'Administrators'

# UAC consent behavior (1 = prompt consent on secure desktop for admins — classic default-ish)
Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System' |
  Select-Object EnableLUA, ConsentPromptBehaviorAdmin, ConsentPromptBehaviorUser,
                PromptOnSecureDesktop
```

**Expect**

- `EnableLUA` → `1`
- Day-to-day user **not** permanently in Administrators (LAPS-managed account may be)
- Consent prompts not set to never notify (`ConsentPromptBehaviorAdmin` should not be `0` for a hardened lab)

---

## Quick pass / fail checklist

| Check | Pass when |
|-------|-----------|
| Defender RTP | Real-time on |
| Firewall | All three profiles enabled |
| BitLocker | OS volume protected + key visible in Entra |
| LAPS | Policy applied + password retrievable from Entra |
| Update ring | Device in pilot ring; deferrals match design |
| Admin hygiene | Standard user daily; UAC enabled |

Record results in your lab notes (not in this public repo if they include hostnames tied to a real tenant). An EXAMPLE compliance CSV shape lives at [`../examples/compliance-checklist.csv`](../examples/compliance-checklist.csv).
