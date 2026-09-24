# BitLocker — Endpoint security Disk encryption (template)

Recreate via **Endpoint security → Disk encryption → Create Policy → BitLocker**.

> Template for a lab baseline. Not a production export. Confirm TPM, UEFI, Secure Boot, WinRE, and Entra join before silent encryption.

| Setting theme | Lab value | Why |
|---------------|-----------|-----|
| Require Device Encryption | Enabled | Ensures OS volume encryption is required |
| OS drive encryption method | XTS-AES 128 (or 256 if policy/hardware supports and you standardize on it) | Balance of strength vs older hardware support |
| Configure TPM startup PIN | Do not allow (for silent lab) | PIN blocks silent enablement |
| Configure TPM startup key | Do not allow (for silent lab) | Same as above |
| Configure TPM startup | Require TPM (or Allow TPM) | Needs TPM present |
| Save BitLocker recovery information to Microsoft Entra ID | Enabled | Helpdesk can retrieve keys |
| Store recovery information in Entra before enabling BitLocker | Required when available | Avoid encrypting without escrow |
| Client-driven recovery password rotation | Enable on Entra joined (lab) | Supports key rotation remote action |
| Allow Warning For Other Disk Encryption | Disabled **only** after inventory shows no third-party encryption | Silent path; dangerous if other crypto exists |
| Allow Standard User Encryption | Enabled if users are standard | Required for silent encryption without local admin |

## Sample JSON shape (template — not Graph export)

```json
{
  "_meta": {
    "title": "BitLocker OS drive — lab template",
    "note": "Recreate in Endpoint security Disk encryption. No tenant IDs."
  },
  "requireDeviceEncryption": true,
  "entraRecoveryPasswordEscrow": true,
  "storeRecoveryInfoBeforeEnable": true,
  "tpmStartupPin": "doNotAllow",
  "tpmStartupKey": "doNotAllow",
  "allowWarningForOtherDiskEncryption": false,
  "allowStandardUserEncryption": true
}
```

## Verify

See [`../docs/VERIFY.md`](../docs/VERIFY.md) (`manage-bde -status`, Intune Recovery keys blade). Pair with [helpdesk-graph-toolkit](https://github.com/SudoShad/helpdesk-graph-toolkit) for scripted key lookup practice once keys exist in Entra.
