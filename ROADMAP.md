# Project Roadmap — dMSA BadSuccessor Detection Lab

Goal: build and **validate** detection content for the dMSA abuse attack family
(BadSuccessor + the Ouroboros variant) in a reproducible lab, since no mainstream
EDR/SIEM ships detection for it yet. Final deliverable: a working detection
capability + a technical write-up.

> The offense and defense halves are coupled by **validation**: a detection is
> only proven by running a real attack against the lab and confirming the rule
> fires. Expect to move back and forth between attacking and detecting, not
> straight down the list.

---

## Phase 0 — Foundations & setup

Get the knowledge and tooling in place before building anything.

- [ ] Read the source disclosures: Akamai "BadSuccessor" (2025) and Huntress "dMSA Ouroboros" (2026). Note the abused attributes.
- [ ] Understand the concepts: dMSA vs gMSA vs regular service account; Kerberos TGT/TGS and the PAC; AD OUs and ACL delegation.
- [ ] Pick a hypervisor (Hyper-V / VMware / VirtualBox / Proxmox) with enough RAM/disk for a DC + a client VM.
- [ ] Obtain a Windows Server 2025 evaluation ISO and a Windows client ISO.
- [x] Repo workflow set up (fork + upstream, branches, PRs). ✅

**Exit criteria:** can explain the attack in plain English; working VMs ready to build on.

---

## Phase 1 — Vulnerable lab → `lab-setup/`

- [ ] Install + promote a Server 2025 Domain Controller; record OS build/patch level.
- [ ] Create an OU and delegate **write** permissions over it to an ordinary user (the BadSuccessor pre-condition).
- [ ] Create a privileged target account (e.g. a Domain Admin) to "succeed."
- [ ] Turn on the audit logging the detection phase needs.
- [ ] Snapshot a clean state + write a reset procedure.

**Exit criteria:** a documented, repeatable lab where a low-priv user holds the abusable permissions, with logging on.

---

## Phase 2 — Attack reproduction → `attack-chain/`

- [ ] Reproduce **BadSuccessor** (original): create/modify dMSA → link to privileged account → obtain a ticket with its privileges.
- [ ] Reproduce **Ouroboros** (variant): same attributes, different sequence; document the ordering carefully.
- [ ] For each run, keep an **attack log**: timestamp → action → resulting artifact. This is the ground truth detection maps against.

**Exit criteria:** both techniques reproducibly succeed in the lab, with attack logs.

---

## Phase 3 — Telemetry pipeline → `detection-rules/`

- [ ] Deploy Sysmon on the DC; start a tuned config.
- [ ] Identify the relevant event IDs (object change 4662/5136, Kerberos 4768/4769, dMSA-related Directory Service events).
- [ ] Stand up a SIEM (Wazuh or Splunk free) and ship Windows + Sysmon logs into it.

**Exit criteria:** events from the lab are searchable in a SIEM.

---

## Phase 4 — Detection content → `detection-rules/`  *(core deliverable)*

Driven by Phase 2 attack runs. Loop: attack → collect events → write rule → re-run to confirm fire → run normal activity to confirm no false positive.

- [ ] Document the **correlation logic** (which events, in what sequence) for each technique.
- [ ] Write **Sigma** rules for BadSuccessor.
- [ ] Write **Sigma** rules for Ouroboros (the rarer, higher-value one — few public sources cover this).
- [ ] Translate Sigma → Splunk and/or Wazuh.
- [ ] Build a **validation table**: attack run → events seen → rule → fired? (TP) / clean run → no fire? (FP).

**Exit criteria:** rules that reliably fire on both attacks and stay quiet on normal admin activity.

---

## Phase 5 — Proactive enumeration tool → `enumeration/`

- [ ] Read-only PowerShell script that enumerates OU ACLs and flags principals who can create/modify dMSAs.
- [ ] Extend it to cover the attributes the Ouroboros variant touches (beyond what Akamai's tool checks).
- [ ] Produce sample output from the lab + remediation notes per finding.

**Exit criteria:** point it at the lab domain and get an accurate "who is exposed" report.

---

## Phase 6 — End-to-end validation

- [ ] Reset lab to clean snapshot, run each attack, confirm every detection fires.
- [ ] Run a batch of legitimate admin actions, confirm no false positives.
- [ ] Run the enumeration tool, confirm it flags the lab's intentional exposure.
- [ ] Iterate on any rule that misses or over-fires.

**Exit criteria:** documented, repeatable proof the detection works.

---

## Phase 7 — Write-up & portfolio → `writeup.md`

- [ ] Fill in `writeup.md`: background, lab, attack reproduction, detection, enumeration, findings/limitations, references.
- [ ] Add a lab diagram and screenshots (tickets obtained, rules firing in the SIEM).
- [ ] Map detections to MITRE ATT&CK where relevant.
- [ ] Note the open Huntress ↔ Microsoft MSRC dispute and what it means for defenders.
- [ ] Final polish: clean READMEs, clear "how to reproduce" instructions.

**Exit criteria:** a self-contained write-up someone else could follow, suitable for a portfolio showcase.

---

## Flow at a glance

```
Phase 0  foundations & setup
   │
   ▼
Phase 1  vulnerable lab ──────────────┐
   │                                  │
   ▼                                  ▼
Phase 2  attack reproduction     Phase 3  telemetry pipeline
   │                                  │
   └──────────────┬───────────────────┘
                  ▼
            Phase 4  detection content  (core)
                  │
   Phase 5  enumeration tool (can run in parallel)
                  │
                  ▼
            Phase 6  end-to-end validation
                  │
                  ▼
            Phase 7  write-up & portfolio
```

> Timelines depend on how fast the concepts click; adjust as you go. Track
> progress by checking off the boxes above in your PRs.
