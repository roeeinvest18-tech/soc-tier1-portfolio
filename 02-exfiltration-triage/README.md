# Network Exfiltration Triage — Packet Capture Analysis

## Scenario

A packet capture from a SOC training exercise was escalated with a single note: "Multiple suspicious activities. Triage." This case study covers the triage workflow for one identified actor in that capture — from a whole-capture traffic inventory to a classified finding ready for handoff.

## Initial Evidence

Working from the guidance to "read summaries before packets," the capture's TCP conversations were reviewed via **Statistics → Conversations → TCP**, sorted by byte volume — a fast way to surface an outsized transfer against the rest of the capture.

## Investigation

Alert → get the traffic inventory → sort by byte volume → isolate the outlier stream → inspect its content → classify → extract IOCs → rate severity:

1. Opened **Statistics → Conversations → TCP** and sorted by bytes transferred, rather than reading packets line by line.
2. One conversation stood out for its volume relative to the rest of the capture, involving the internal host and external IP `203.0.113.99`.
3. Isolated that single conversation with `tcp.stream eq 131` to inspect it in detail.
4. In the packet Info column (frame 754), found a reference to `upload.file-anon.com` — a domain name pattern consistent with an external file-upload/anonymization service.
5. Classified the session as **suspicious outbound transfer consistent with exfiltration**: an internal host sending a large volume of encrypted data outbound to an external file-upload-style host on port 443.

## Queries

See `queries.txt`.

## Findings

- **Confirmed:** a large-volume outbound TCP session (`tcp.stream 131`) from an internal host to `203.0.113.99` (`upload.file-anon.com`) on port 443.
- **Observed:** the destination domain name pattern (`file-anon`) is consistent with an external file-drop/anonymized-upload service rather than an approved business destination based on the available capture context.
- **Cannot be determined from this capture alone:** exact byte count, the specific file(s) transferred, or whether the destination is a known-bad IOC versus an unsanctioned-but-legitimate personal file-sharing use. Traffic was encrypted, so content could not be inspected directly.

## Conclusion

The evidence supports a **suspicious** large-volume outbound transfer consistent with data exfiltration to an untrusted external host. It does not by itself confirm malicious intent — that distinction would require additional context such as asset ownership, business justification, or DLP/proxy logs for the destination.

## Tier 1 Decision

**Escalate**, self-rated severity 4/5 by the original analyst (1 = needs immediate action, 5 = informational) — reflecting a credible exfiltration indicator worth prompt follow-up, though not necessarily an active, in-progress critical incident at the time of review. Escalation package: source host, destination IP/domain, port, and the `tcp.stream 131` reference so IR/DLP teams can pull the full session and determine data sensitivity and intent.

## Skills Demonstrated

- Statistics-first triage using conversations and byte volume to identify outliers
- Isolating a specific TCP stream for focused inspection
- Extracting network indicators from packet data
- Distinguishing what a capture can and cannot confirm when traffic is encrypted
- Severity assessment and escalation handoff

## MITRE ATT&CK

**T1567 — Exfiltration Over Web Service.** The observed behavior (large outbound transfer over HTTPS to an external file-upload-style domain) matches this technique at a high level; the capture does not identify the specific web service, so no more specific sub-technique is claimed.
