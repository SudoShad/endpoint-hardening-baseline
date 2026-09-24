# Endpoint Hardening Baseline

**Author:** Shadman Bari · [shadman.io](https://shadman.io) · [LinkedIn](https://linkedin.com/in/shadman-bari) · shadman@shadman.io  
**Focus:** Desktop Support / IT Support / Jr Sysadmin — Windows endpoint hardening as **documented Intune / policy-as-code**  
**Companions:** [helpdesk-graph-toolkit](https://github.com/SudoShad/helpdesk-graph-toolkit) · [monitoring-breakglass-ops](https://github.com/SudoShad/monitoring-breakglass-ops) · [linux-homelab](https://github.com/SudoShad/linux-homelab) · [linux-ops-toolkit](https://github.com/SudoShad/linux-ops-toolkit) · [ad-intune-mini-tenant](https://github.com/SudoShad/ad-intune-mini-tenant) *(PARKED)*

CIS-inspired Windows endpoint hardening expressed as Intune Settings Catalog / Endpoint security documentation, markdown settings tables, and sample JSON / OMA-URI **templates**. No live tenant is required to read or interview from this repo.

> Lab / trial tenant only. Do **not** blast these settings at production without change control, pilot rings, and rollback. Samples are templates to recreate — not exported production policies and not fake tenant GUIDs.

---

## Problem

Helpdesk and junior sysadmin interviews expect more than “I can reset a password.” Employers want evidence you understand **why** endpoints get locked down and **how** those controls land in Microsoft Intune:

- Microsoft Defender Antivirus (real-time, cloud-delivered, PUA)
- Windows Firewall across domain / private / public profiles
- BitLocker OS-drive encryption with recovery key escrow to Entra
- Windows LAPS for rotatable local admin passwords
- Conservative Windows Update rings that won’t break deskside support
- Local admin / UAC hygiene (documented at a practical level)

Most “hardening” GitHub repos dump GPO screenshots or claim a production export. This one is honest policy-as-code documentation you can recreate in a trial Intune tenant or talk through in an interview.

---

## Baseline pillars

| Pillar | Goal | Detail |
|--------|------|--------|
| Defender Antivirus | Catch malware early | Real-time protection, cloud-delivered protection, Potentially Unwanted App (PUA) protection |
| Firewall | Reduce lateral / inbound noise | Domain, private, and public profiles enabled with sensible inbound defaults |
| BitLocker | Protect data at rest | OS drive encryption; recovery password stored in Microsoft Entra ID |
| Windows LAPS | Kill shared local-admin passwords | Rotate built-in (or managed) local admin; back up password to Entra via Intune |
| Update rings | Patch without chaos | Defer quality/feature updates enough for helpdesk to react — not so long you are exposed |
| Local admin / UAC | Shrink blast radius | Prefer standard users; document UAC elevation expectations (high level) |

Full rationale: [`docs/BASELINE.md`](docs/BASELINE.md).

---

## What’s in the repo

| Path | Purpose |
|------|---------|
| [`docs/BASELINE.md`](docs/BASELINE.md) | Pillars + why each control matters for deskside / Jr Sysadmin work |
| [`docs/RUNBOOK.md`](docs/RUNBOOK.md) | Intune admin center click-paths (or import JSON where provided) |
| [`docs/VERIFY.md`](docs/VERIFY.md) | Endpoint verification commands for a lab Windows VM |
| [`policies/`](policies/) | Settings tables + sample JSON / OMA-URI templates |
| [`examples/compliance-checklist.csv`](examples/compliance-checklist.csv) | **EXAMPLE** checklist shape only |

---

## How to use in Intune

1. Use a **Microsoft 365 trial / lab tenant** (or the parked [ad-intune-mini-tenant](https://github.com/SudoShad/ad-intune-mini-tenant) path when you have bandwidth).
2. Enroll a Windows 11 lab VM (Entra join + Company Portal / Autopilot as you prefer).
3. Recreate each control from [`docs/RUNBOOK.md`](docs/RUNBOOK.md) — Endpoint security blades first, Settings Catalog where noted.
4. Optionally adapt the JSON snippets under [`policies/`](policies/) as reference when building Settings Catalog / custom OMA-URI profiles. Treat them as **templates to recreate**, not drop-in tenant exports.
5. Assign to a **pilot** device group first. Verify with [`docs/VERIFY.md`](docs/VERIFY.md) before any broader assignment.

---

## Safety

- **Lab / trial only** until you have change tickets, pilots, and rollback.
- Silent BitLocker and LAPS can lock you out of recovery paths if escrow fails — always confirm Entra key / password backup before forcing encryption or rotating admins.
- Update deferrals that are too aggressive leave you unpatched; too lax floods helpdesk with feature-update surprises. The rings here are intentionally conservative and helpdesk-friendly.
- No secrets, tenant IDs, recovery keys, or real device exports belong in this repo.

---

## Resume bullet (paste-ready)

> Documented a public **CIS-inspired Windows endpoint hardening baseline** (`endpoint-hardening-baseline`) as Intune / policy-as-code — Defender Antivirus, Firewall, BitLocker with Entra key escrow, Windows LAPS, and conservative Update rings — with admin-center runbooks and endpoint verification steps for Desktop Support / Jr Sysadmin interviews (lab/trial only; no production blast).

---

## Skills demonstrated

- Microsoft Intune Endpoint security + Settings Catalog literacy  
- Windows Defender Antivirus, Firewall, BitLocker, LAPS, Update rings  
- Policy-as-code documentation (tables, JSON / OMA-URI templates)  
- Helpdesk-safe change control (pilot rings, verification, rollback notes)  
- Honest portfolio practice (templates marked as such; no fake exports)

---

## Related labs

| Repo | Role |
|------|------|
| [helpdesk-graph-toolkit](https://github.com/SudoShad/helpdesk-graph-toolkit) | PowerShell + Graph helpdesk automation (BitLocker key lookup, stale devices, password reset) |
| [monitoring-breakglass-ops](https://github.com/SudoShad/monitoring-breakglass-ops) | Monitoring alert triage + break-glass / emergency access patterns |
| [linux-homelab](https://github.com/SudoShad/linux-homelab) | Proxmox + AD/GPO + WireGuard + osTicket + Wazuh |
| [linux-ops-toolkit](https://github.com/SudoShad/linux-ops-toolkit) | POSIX `sh` backup / SSH guard / health check |
| [ad-intune-mini-tenant](https://github.com/SudoShad/ad-intune-mini-tenant) | Entra + Intune enroll lab — **PARKED** |

---

## License

MIT © 2026 Shadman Bari — see [LICENSE](LICENSE).
