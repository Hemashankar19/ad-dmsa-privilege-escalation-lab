# 02 — Domain controller promotion

Promote DC01 (Server 2025, build 26100, configured in
[`01-host-setup.md`](01-host-setup.md)) into the first DC of a new forest
`dmsalab.local`.

## Install the AD DS role

Elevated PowerShell on DC01:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

## Promote to a new forest

```powershell
Install-ADDSForest -DomainName "dmsalab.local" -DomainNetbiosName "DMSALAB" -InstallDNS -Force
```

- Prompts for a **SafeModeAdministratorPassword** (DSRM recovery password).
  It must meet complexity (upper + lower + digit + symbol, 8+ chars) — a weak
  value fails prerequisite verification with
  *"Directory Services Restore Mode password does not meet the password
  complexity requirements."*
- The DNS delegation warning ("authoritative parent zone cannot be found") is
  **expected** in an isolated lab with no parent DNS — no action needed.
- The server reboots automatically when promotion completes.

### Gotchas hit

- **Single dash on parameters.** `-InstallDNS`, not `--InstallDNS`. A double
  dash makes PowerShell treat it as a positional argument:
  *"A positional parameter cannot be found that accepts argument '--InstallDNS'."*
- **DSRM password complexity** — see above.

## Verify (after reboot)

Log in as **DMSALAB\Administrator** (the box is now a domain member):

```powershell
Get-ADDomain | Select-Object DNSRoot, NetBIOSName, DomainMode
Get-ADForest | Select-Object Name, ForestMode
```

Observed on this build:

| Property     | Value               |
|--------------|---------------------|
| DNSRoot      | dmsalab.local       |
| NetBIOSName  | DMSALAB             |
| DomainMode   | Windows2025Domain   |
| ForestMode   | Windows2025Forest   |

A single Server 2025 DC comes up at the **Windows2025** domain/forest
functional level, so the dMSA object class and the KDC logic BadSuccessor
abuses are fully available. (A 2025 functional level is not strictly required
for dMSA — a 2025 DC with the 2025 schema suffices — but a fresh single-DC
forest lands there anyway.)

## Status

- [x] AD DS role installed
- [x] Promoted to forest `dmsalab.local` (NetBIOS `DMSALAB`)
- [x] Verified: Windows2025 domain/forest functional level
- [ ] OU `dMSA_OU` + `attacker` (low-priv) and `da_target` (Domain Admin) accounts
- [ ] CC-only delegation on the OU to `attacker` (the BadSuccessor precondition)
- [ ] Audit policy + SACL so 5136/5137/4662/4768/4769 fire
- [ ] Snapshot `Phase1-clean-preattack`
