# Windows Firewall — settings table (template)

Recreate via **Endpoint security → Firewall → Create Policy → Windows Firewall**, or Settings Catalog equivalents.

> Template for a lab baseline. Not a production export.

| Setting | Domain | Private | Public | Notes |
|---------|--------|---------|--------|-------|
| Firewall enabled | Yes | Yes | Yes | All profiles on |
| Default inbound | Block | Block | Block | Allow established/related via stateful firewall |
| Default outbound | Allow | Allow | Allow | Tighten later with app rules if needed |
| Notify user when inbound blocked | Optional | Optional | Optional | Useful in lab for visibility |
| Stealth mode | Allowed / On | Allowed / On | Allowed / On | Where exposed in profile |

## OMA-URI style references (custom profiles — optional)

Only needed if you are demonstrating custom OMA-URI fluency. Prefer the Endpoint security Firewall profile when available.

| Concept | CSP-oriented path (illustrative) |
|---------|----------------------------------|
| Domain profile enable | `./Vendor/MSFT/Firewall/MdmStore/DomainProfile/EnableFirewall` |
| Private profile enable | `./Vendor/MSFT/Firewall/MdmStore/PrivateProfile/EnableFirewall` |
| Public profile enable | `./Vendor/MSFT/Firewall/MdmStore/PublicProfile/EnableFirewall` |

Values are typically boolean / integer per CSP docs — confirm against current [Firewall CSP](https://learn.microsoft.com/en-us/windows/client-management/mdm/firewall-csp) before building custom OMA-URI profiles.

## Helpdesk tip

If a business app breaks after enablement, collect remote address/port and create a **scoped** allow rule — do not disable the Public profile on laptops that leave trusted networks.
