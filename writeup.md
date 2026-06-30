# Technical Write-up — dMSA BadSuccessor Detection

> Final deliverable. Fill this in as the project progresses. The outline below is a starting skeleton — adjust freely.

## 1. Summary

_One paragraph: what the attack family is, what you built, and the headline result (working detection for a technique that currently has none)._

## 2. Background

- What dMSAs are and why Server 2025 introduced them.
- BadSuccessor (Akamai, 2025) — the original technique and the attributes it abuses.
- dMSA Ouroboros (Huntress, 2026) — the variant and why it's harder (survives rotation/deletion, defeats Credential Guard, undetected by current tooling).
- The open Huntress ↔ Microsoft MSRC dispute, and why that means defensive guidance is still unsettled.

## 3. Lab Environment

_Reference `/lab-setup`. Diagram of the lab, OS/patch levels, the delegation that creates the vulnerable condition._

## 4. Attack Reproduction

_Reference `/attack-chain`. Step-by-step for each technique, with the artifacts produced (tickets, modified objects)._

## 5. Detection

_Reference `/detection-rules`. The events, the correlation logic, the rules, and the validation results (TP/FP table)._

## 6. Proactive Enumeration

_Reference `/enumeration`. How defenders find exposed OUs before exploitation._

## 7. Findings & Limitations

_What the detection catches, what it misses, false-positive notes, and what remains open._

## 8. References

- Akamai — BadSuccessor disclosure (2025)
- Huntress — dMSA Ouroboros disclosure (2026)
- _(add links, MITRE ATT&CK mappings, event ID docs as you collect them)_

## Disclaimer

For defensive security research and education only. All testing performed in an isolated lab with no connection to production systems.
