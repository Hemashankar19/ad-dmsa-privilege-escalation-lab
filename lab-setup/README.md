# lab-setup

**Owner:** Hema (offense / lab builder)

Scripts and docs to stand up a deliberately vulnerable Windows Server 2025 Active Directory lab that reproduces the dMSA BadSuccessor conditions.

## Goal

A repeatable lab where a *low-privileged* user holds the OU permissions that BadSuccessor abuses, so the attack in `/attack-chain` can be demonstrated and the detections in `/detection-rules` can be validated against real telemetry.

## What to build here

1. **Domain controller** — a Server 2025 DC (this feature only exists on 2025). Note the build/patch level, because Microsoft patched the original bidirectional-link issue but not the whole family.
2. **OU + delegation** — an OU with a normal user granted write permissions over it (e.g. `Create child objects` / write on dMSA-related attributes). This is the "91% of environments already have this" condition.
3. **A privileged target account** — e.g. a Domain Admin the attacker will try to "succeed."
4. **Logging baseline** — make sure the DC is generating the events the detection side needs (see `/detection-rules`). Coordinate the audit policy here so detection has data to work with.

## What to learn first

- What a dMSA is and how it differs from a gMSA / regular service account.
- The two attributes BadSuccessor abuses (research the Akamai write-up — `msDS-ManagedAccountPrecededByLink` and `msDS-DelegatedMSAState`).
- How OU delegation / ACLs work in AD (`dsacls`, ACL inheritance).
- How to snapshot/checkpoint VMs so you can reset the lab between test runs.

## Deliverables

- [ ] Documented step-by-step build (so anyone can reproduce it)
- [ ] Provisioning notes/scripts (PowerShell)
- [ ] A "reset to clean state" procedure
- [ ] Confirmation the required audit logging is on

> Keep secrets, license keys, and VM images **out** of git. See the repo `.gitignore`.
