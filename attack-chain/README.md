# attack-chain

**Owner:** Hema (offense / lab builder)

Automation that reproduces the dMSA abuse attack family end-to-end, against the lab in `/lab-setup`. Each attack run is what the detection side uses to confirm their rules actually fire.

## Scope

Two techniques, kept separate so detection can be validated against each:

- `badsuccessor/` — the original technique (Akamai, 2025): create/modify a dMSA, link it to a privileged account, get a Kerberos ticket with that account's privileges.
- `ouroboros/` — the variant (Huntress, 2026): same underlying attributes, different manipulation sequence; self-sustaining credential extraction that survives password rotation and account deletion.

## What to build here

For **each** technique:
1. A script (or documented manual steps) that performs the attack from the low-privileged user's context.
2. A clear record of *what changed in AD* and *what tickets/credentials were obtained* — this is the ground truth detection compares against.
3. Notes on **timing and order of operations** (especially for Ouroboros, where the sequence is the whole point).

## What to learn first

- Kerberos basics: TGT vs TGS, PAC (the privilege data inside a ticket).
- How tools like Rubeus / the public BadSuccessor PoC request tickets for a dMSA.
- The difference between the patched bidirectional-link path and the variant path.

## Deliverables

- [ ] Working `badsuccessor` reproduction
- [ ] Working `ouroboros` reproduction
- [ ] An "attack log" per run (timestamp, action, resulting artifact) the detection side can map to events

> ⚠️ Lab-only. This must never be pointed at any system you don't own. The repo disclaimer and isolation note apply to everything here.
