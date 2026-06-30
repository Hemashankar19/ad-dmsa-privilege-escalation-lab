# detection-rules

**Owner:** Charan (defense / detection content)

The core deliverable: detection logic for the dMSA abuse family. No mainstream EDR/SIEM ships this out of the box — that gap is the whole point of the project.

## What to build here

Suggested layout (create these as you go):

- `sysmon/` — Sysmon config tuned to capture the relevant process/LDAP/Kerberos activity.
- `sigma/` — vendor-neutral [Sigma](https://github.com/SigmaHQ/sigma) rules (the portable, portfolio-friendly format).
- `splunk/` and/or `wazuh/` — the Sigma rules translated into a SIEM you actually run in the lab.
- `event-correlation.md` — which Windows Security / Directory Service event IDs, in what sequence, indicate the attack (e.g. dMSA object creation + the link attribute being set + a TGS request for the dMSA).

## How to work

Detection is **driven by the attack-chain**. The loop is:
1. Hema runs an attack in `/attack-chain`.
2. You collect the events it generated on the DC.
3. You write a rule that catches that pattern.
4. Re-run the attack → confirm the rule fires (true positive).
5. Run normal admin activity → confirm it does **not** fire (false positive check).

## What to learn first

- Windows event IDs for object changes (4662, 5136) and Kerberos (4768/4769).
- Sigma rule syntax and how `sigma convert` produces Splunk/Wazuh queries.
- The idea of *correlation* (a sequence of events), not just single-event matching — the variant specifically hides in "normal-looking" individual actions.

## Deliverables

- [ ] Sysmon config
- [ ] Sigma rule set covering BadSuccessor **and** Ouroboros
- [ ] At least one SIEM translation (Splunk or Wazuh)
- [ ] A validation table: attack run → events seen → rule → fired? (TP/FP)
