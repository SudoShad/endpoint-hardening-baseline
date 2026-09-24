# Windows LAPS — Entra / Intune (template)

Recreate via **Endpoint security → Account protection → Create Policy → Local admin password solution (Windows LAPS)**.

Setting names follow the Windows LAPS CSP / Microsoft Learn policy reference (`BackupDirectory`, `PasswordAgeDays`, etc.).

> Template for a lab baseline. Not a production export. No tenant GUIDs.

| CSP / setting name | Lab value | Notes |
|--------------------|-----------|-------|
| BackupDirectory | `1` (Microsoft Entra ID) | `0` = disabled, `2` = Active Directory only |
| PasswordAgeDays | `30` | Entra backup minimum is **7** days |
| PasswordComplexity | `4` | Large + small + numbers + special |
| PasswordLength | `14` | Raise if local password policy allows |
| PostAuthenticationResetDelay | `24` | Hours before post-auth actions (0 disables) |
| PostAuthenticationActions | `3` | Reset password and sign out |
| AdministratorAccountName | *(omit)* | Omit to manage built-in Administrator; set only for a pre-created custom account |
| AutomaticAccountManagementEnabled | `0` (lab default) | Optional newer Windows 11 24H2+ feature — enable only if you document the managed account name |

## Sample OMA-URI style snippets (custom profile — optional)

Prefer the Account protection LAPS profile UI. Custom OMA-URI is for demos / edge cases. Confirm paths against current [LAPS CSP](https://learn.microsoft.com/en-us/windows/client-management/mdm/laps-csp) docs before use.

| Setting | Illustrative OMA-URI | Example value |
|---------|----------------------|---------------|
| BackupDirectory | `./Device/Vendor/MSFT/LAPS/Policies/BackupDirectory` | `1` |
| PasswordAgeDays | `./Device/Vendor/MSFT/LAPS/Policies/PasswordAgeDays` | `30` |
| PasswordComplexity | `./Device/Vendor/MSFT/LAPS/Policies/PasswordComplexity` | `4` |
| PasswordLength | `./Device/Vendor/MSFT/LAPS/Policies/PasswordLength` | `14` |

Data type is typically Integer for these CSP nodes — verify in Learn docs when building a custom profile.

## Sample JSON shape (template)

```json
{
  "_meta": {
    "title": "Windows LAPS Entra backup — lab template",
    "note": "Recreate in Endpoint security Account protection. Not a live export."
  },
  "BackupDirectory": 1,
  "PasswordAgeDays": 30,
  "PasswordComplexity": 4,
  "PasswordLength": 14,
  "PostAuthenticationResetDelay": 24,
  "PostAuthenticationActions": 3
}
```

## Helpdesk tip

After first rotation, prove you can read the password from Entra / Intune for that device, then rotate again. Never paste LAPS passwords into public repos or long-lived ticket comments.
