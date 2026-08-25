# CV Project Summary

- Investigated ARP cache poisoning / man-in-the-middle activity using Wireshark packet analysis, constructing display filters to distinguish legitimate unicast ARP replies from unsolicited spoofed replies, mapping affected hosts, and reaching an escalation decision.
- Triaged a network capture for data exfiltration using conversation- and byte-volume analysis in Wireshark, isolating a suspicious high-volume outbound session, extracting IOCs, and rating severity for handoff.
- Tuned Splunk (SPL) detection rules for false-positive reduction across multiple scenarios (egress volume, mass process execution, lockout storms), documenting the specific false-negative or dependency risk each tuning decision introduced.
