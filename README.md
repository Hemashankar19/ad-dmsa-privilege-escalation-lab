# dMSA BadSuccessor Detection Lab

Research project reproducing and building detection content for the "BadSuccessor" family of Active Directory privilege escalation techniques affecting delegated Managed Service Accounts (dMSAs) on Windows Server 2025, including the original technique disclosed by Akamai (2025) and the "dMSA Ouroboros" variant disclosed by Huntress (2026).

## Background

Windows Server 2025 introduced delegated Managed Service Accounts (dMSAs) as a safer replacement for traditional service accounts. A flaw in how dMSA "migration" linking works allows a user with ordinary, commonly-delegated OU permissions to create or modify a dMSA object and claim it as the successor to a privileged account (e.g. a Domain Admin), causing the KDC to issue Kerberos tickets carrying that account's full privileges.

No mainstream EDR/SIEM currently ships out-of-the-box detection for this attack family. This project aims to close that gap with a reproducible lab, attack automation, and a published detection rule set.

## Project Structure

- `/lab-setup` — scripts and docs to stand up a vulnerable Server 2025 AD lab with realistic OU delegation
- `/attack-chain` — automation reproducing BadSuccessor and the Ouroboros variant
- `/detection-rules` — Sigma rules and Wazuh/Splunk equivalents for detecting dMSA abuse
- `/enumeration` — PowerShell script to flag OUs/principals with exploitable permissions before exploitation
- `/writeup.md` — full technical write-up, findings, and screenshots

## Status

 Work in progress.

## Contributors

- [Muni Hema Shankar](https://github.com/Hemashankar19)
- [Ankam Charan Teja](https://github.com/ankamteja)

## References

- Akamai, "BadSuccessor" research disclosure (2025)
- Huntress, "dMSA Ouroboros" research disclosure (2026)

## Disclaimer

This project is for defensive security research and education only. All testing was performed in an isolated lab environment with no connection to production systems.
