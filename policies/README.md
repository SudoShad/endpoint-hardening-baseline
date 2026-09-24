# Policies — Intune-oriented artifacts

These files document a **lab hardening baseline**. They are templates to recreate in Microsoft Intune — **not** exports from a production tenant.

| File | Covers |
|------|--------|
| [`defender.json`](defender.json) | Microsoft Defender Antivirus settings as a JSON template + comments in `_meta` |
| [`firewall.md`](firewall.md) | Windows Firewall profile settings table |
| [`bitlocker.md`](bitlocker.md) | BitLocker / Disk encryption settings table (Endpoint security) |
| [`laps.md`](laps.md) | Windows LAPS (Entra backup) settings table + sample OMA-URI style notes |
| [`update-rings.md`](update-rings.md) | Conservative Windows Update ring values |

## Honesty rules

- No tenant GUIDs, device IDs, or real assignment group IDs.
- No claim that JSON was exported from Intune Graph unless you replace these with your own export later.
- Portal labels change; if a setting name differs, search Intune Settings Catalog / Endpoint security for the closest match and update your lab notes.

## Suggested recreate order

1. Defender Antivirus  
2. Firewall  
3. BitLocker (after TPM / Entra join checks)  
4. Windows LAPS  
5. Update rings  
6. Local admin / UAC documentation (see [`../docs/BASELINE.md`](../docs/BASELINE.md))
