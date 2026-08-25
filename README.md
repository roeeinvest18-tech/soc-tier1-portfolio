# SOC Tier 1 Case Studies

Three hands-on packet- and SIEM-based investigations demonstrating practical SOC skills: alert triage, evidence-based investigation, technical analysis, and escalation judgment.

- [`01-arp-spoofing-mitm/`](01-arp-spoofing-mitm/README.md) — ARP cache poisoning / gateway impersonation, detected via Wireshark.
- [`02-exfiltration-triage/`](02-exfiltration-triage/README.md) — Network exfiltration triage from a packet capture, via Wireshark.
- [`03-spl-detection-tuning/`](03-spl-detection-tuning/README.md) — Splunk (SPL) detection rule tuning, with documented false-negative trade-offs.

Each folder contains a `README.md` (scenario, investigation steps, findings, Tier 1 decision, skills demonstrated), a `queries.txt` with the filters/queries used, and `INTERVIEW_NOTES.md` for talking through the case out loud.

See [`CV_PROJECT_SUMMARY.md`](CV_PROJECT_SUMMARY.md) for CV-ready bullet points summarizing all three.
