# Project Roadmap — dMSA BadSuccessor Detection Lab

Goal: build and **validate** detection content for the dMSA abuse attack family
(BadSuccessor + the Ouroboros variant) in a reproducible lab, since no mainstream
EDR/SIEM ships detection for it yet. Final deliverable: a working detection
capability + a technical write-up.

**Team split**
- **Hema** — offense: vulnerable Server 2025 lab + attack automation (`lab-setup/`, `attack-chain/`)
- **Charan** — defense: detection content + enumeration tool (`detection-rules/`, `enumeration/`)

> The two halves are coupled by **validation**: detection can only be proven by
> running real attacks against the lab and confirming the rules fire. Plan to work
> in tight loops, not in isolation.

---

## Phase 0 — Foundations & setup _(both, ~1 week)_

Get the knowledge and tooling in place before building anything.

- [ ] Read the source disclosures: Akamai "BadSuccessor" (2025) and Huntress "dMSA Ouroboros" (2026). Take notes on the abused attributes.
- [ ] Understand the concepts: dMSA vs gMSA vs regular service account; Kerberos TGT/TGS and the PAC; AD OUs and ACL delegation.
- [ ] Pick a hypervisor (Hyper-V / VMware / VirtualBox / Proxmox) and confirm you have the RAM/disk for a DC + a client VM.
- [ ] Obtain a Windows Server 2025 evaluation ISO and a Windows client ISO.
- [x] Repo workflow set up (fork + upstream, branches, PRs). ✅
- [ ] Agree on a shared note-keeping place (this repo) and a regular sync cadence.

**Exit criteria:** both teammates can explain the attack in plain English and have working VMs to build on.

---

## Phase 1 — Vulnerable lab _(Hema, ~1 week)_ → `lab-setup/`

- [ ] Install + promote a Server 2025 Domain Controller; record OS build/patch level.
- [ ] Create an OU and delegate **write** permissions over it to an ordinary user (the BadSuccessor pre-condition).
- [ ] Create a privileged target account (e.g. a Domain Admin) to "succeed."
- [ ] Turn on the audit logging the detection side needs (coordinate with Charan).
- [ ] Snapshot a clean state + write a reset procedure.

**Depends on:** Phase 0.
**Exit criteria:** a documented, repeatable lab where a low-priv user holds the abusable permissions, with logging on.

---

## Phase 2 — Attack reproduction _(Hema, ~1–2 weeks)_ → `attack-chain/`

- [ ] Reproduce **BadSuccessor** (original): create/modify dMSA → link to privileged account → obtain a ticket with its privileges.
- [ ] Reproduce **Ouroboros** (variant): same attributes, different sequence; document the ordering carefully.
- [ ] For each run, keep an **attack log**: timestamp → action → resulting artifact. This is the ground truth detection maps against.

**Depends on:** Phase 1.
**Exit criteria:** both techniques reproducibly succeed in the lab, with attack logs.

---

## Phase 3 — Telemetry pipeline _(Charan, ~1 week, can start during Phase 1)_ → `detection-rules/`

- [ ] Deploy Sysmon on the DC; start a tuned config.
- [ ] Identify the relevant event IDs (object change 4662/5136, Kerberos 4768/4769, dMSA-related Directory Service events).
- [ ] Stand up a SIEM (Wazuh or Splunk free) and ship Windows + Sysmon logs into it.

**Depends on:** Phase 1 (needs the DC + logging on). Knowledge prep can start in Phase 0.
**Exit criteria:** events from the lab are searchable in a SIEM.

---

## Phase 4 — Detection content _(Charan, ~2–3 weeks — the core deliverable)_ → `detection-rules/`

Driven by Phase 2 attack runs. Loop: attack → collect events → write rule → re-run to confirm fire → run normal activity to confirm no false positive.

- [ ] Document the **correlation logic** (which events, in what sequence) for each technique.
- [ ] Write **Sigma** rules for BadSuccessor.
- [ ] Write **Sigma** rules for Ouroboros (the rarer, higher-value one — few public sources cover this).
- [ ] Translate Sigma → Splunk and/or Wazuh.
- [ ] Build a **validation table**: attack run → events seen → rule → fired? (TP) / clean run → no fire? (FP).

**Depends on:** Phases 2 + 3.
**Exit criteria:** rules that reliably fire on both attacks and stay quiet on normal admin activity.

---

## Phase 5 — Proactive enumeration tool _(Charan, ~1 week, parallelizable)_ → `enumeration/`

- [ ] Read-only PowerShell script that enumerates OU ACLs and flags principals who can create/modify dMSAs.
- [ ] Extend it to cover the attributes the Ouroboros variant touches (beyond what Akamai's tool checks).
- [ ] Produce sample output from the lab + remediation notes per finding.

**Depends on:** understanding from Phase 0–2; can be built independently of detection.
**Exit criteria:** point it at the lab domain and get an accurate "who is exposed" report.

---

## Phase 6 — End-to-end validation _(both, ~1 week)_

- [ ] Reset lab to clean snapshot, run each attack, confirm every detection fires.
- [ ] Run a batch of legitimate admin actions, confirm no false positives.
- [ ] Run the enumeration tool, confirm it flags the lab's intentional exposure.
- [ ] Iterate on any rule that misses or over-fires.

**Depends on:** Phases 4 + 5.
**Exit criteria:** documented, repeatable proof the detection works.

---

## Phase 7 — Write-up & portfolio _(both, ~1 week)_ → `writeup.md`

- [ ] Fill in `writeup.md`: background, lab, attack reproduction, detection, enumeration, findings/limitations, references.
- [ ] Add a lab diagram and screenshots (tickets obtained, rules firing in the SIEM).
- [ ] Map detections to MITRE ATT&CK where relevant.
- [ ] Note the open Huntress ↔ Microsoft MSRC dispute and what it means for defenders.
- [ ] Final polish: clean READMEs, clear "how to reproduce" instructions.

**Exit criteria:** a self-contained write-up someone else could follow, suitable for a portfolio/freelance showcase.

---

## Dependencies at a glance

```
Phase 0 (both)
   ├─> Phase 1 (Hema: lab) ──> Phase 2 (Hema: attacks) ──┐
   │                                                      ├─> Phase 4 (Charan: detection) ─┐
   └─> Phase 3 (Charan: telemetry) ──────────────────────┘                                ├─> Phase 6 (validate) ─> Phase 7 (write-up)
                                                                                           │
       Phase 5 (Charan: enumeration, parallel) ────────────────────────────────────────────┘
```

## Suggested first moves
- **Hema:** start Phase 1 (the lab gates almost everything).
- **Charan:** start Phase 5 (enumeration) — it's read-only, parallel, and a gentle PowerShell on-ramp — while reading up for Phases 3–4.

> Timelines are rough estimates for beginners working part-time; adjust as you learn. Track progress by checking off the boxes above in PRs.
