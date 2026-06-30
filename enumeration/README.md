# enumeration

**Owner:** Charan (defense / detection content)

A script that finds *exposed* OUs and principals **before** exploitation — i.e. who in the domain already has the permissions BadSuccessor needs. Akamai released something similar; the goal here is to extend it to cover the newer variant's conditions too.

## Goal

Point it at a domain and get a report: "these principals can create/modify dMSAs in these OUs and could perform this attack." This is the proactive, blue-team side — it lets a defender close the hole before anyone exploits it.

## What to build here

- A PowerShell script (AD module) that:
  1. Enumerates OUs and their ACLs.
  2. Flags principals with the write permissions relevant to dMSA creation/linking.
  3. Outputs a readable report (and ideally CSV/JSON for evidence).

## What to learn first

- `Get-ACL` / `dsacls` and how to read AD ACEs in PowerShell.
- Which specific rights matter (research the Akamai enumeration tool as a reference, then map the extra attributes the Ouroboros variant touches).
- How to run safely with least privilege (this is a read-only audit, no changes).

## Deliverables

- [ ] Enumeration script (read-only)
- [ ] Sample output from the lab
- [ ] Notes on what each flagged finding means and how to remediate it

> Read-only by design. It should never modify AD objects.
