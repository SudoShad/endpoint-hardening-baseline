# Windows Update rings — conservative helpdesk-friendly (template)

Recreate via **Devices → Windows updates → Update rings → Create profile** (or the newer Windows quality / feature update experience if your tenant shows it).

> Template for a lab baseline. Not a production export. Pick values, document them, and stick to them for the pilot.

| Setting | Suggested lab value | Rationale |
|---------|---------------------|-----------|
| Servicing channel | General Availability | Avoid Insider on helpdesk-managed endpoints |
| Microsoft product updates | Allow | Office / other Microsoft updates |
| Windows drivers | Allow (lab) | Or block if you stage drivers separately |
| Quality update deferral (days) | **7** | Short cushion for known-bad patches without going unpatched |
| Feature update deferral (days) | **30** | Enough time for smoke-test; use **60** if floors are change-averse |
| Feature update uninstall period (days) | Leave default / enable if offered | Rollback window after a bad feature update |
| Automatic update behavior | Auto install + notify reboot / deadline | Match what your helpdesk can support for reboot messaging |
| Active hours / user engagement | Configure gently | Prevent surprise midday reboots on shared desks |
| Deadline for quality updates | Optional short deadline after deferral | Stops eternal “I’ll reboot later” |

## Pilot vs broad

| Ring | Who | Deferrals |
|------|-----|-----------|
| Pilot | IT + volunteer early adopters | Quality 0–3 days, Feature 0–7 days |
| Broad (lab “production-like”) | Remaining lab devices | Quality 7, Feature 30 (this template) |

## Sample JSON shape (template)

```json
{
  "_meta": {
    "title": "Windows Update ring — helpdesk-friendly lab template",
    "note": "Recreate in Intune Update rings UI. No tenant or group IDs."
  },
  "servicingChannel": "generalAvailability",
  "qualityUpdateDeferralDays": 7,
  "featureUpdateDeferralDays": 30,
  "microsoftProductUpdates": "allow"
}
```

## Helpdesk tip

When a quality update breaks a line-of-business app, pause the ring or move affected devices back to pilot while you document the workaround — do not set feature deferral to the maximum forever as a substitute for testing.
