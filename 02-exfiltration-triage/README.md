# Network Exfiltration Triage — Packet Capture Analysis

## Scenario

A packet capture from a SOC training exercise was escalated with a single note: "Multiple suspicious activities. Triage." This case study covers the triage workflow for one identified actor in that capture — from a whole-capture traffic inventory to a classified finding ready for handoff.

## Initial Evidence

Working from the guidance to "read summaries before packets," the capture's TCP conversations were reviewed via **Statistics → Conversations → TCP**, sorted by byte volume — a fast way to surface an outsized transfer against the rest of the capture.

## Investigation

Alert → get the traffic inventory → sort by byte volume → isolate the outlier stream → inspect available metadata → classify → record training indicators → document escalation rationale:

1. Opened **Statistics → Conversations → TCP** and sorted by bytes transferred, rather than reading packets line by line.
2. One conversation stood out for its volume relative to the rest of the capture, involving the internal host and external IP `203.0.113.99`.
3. Isolated that single conversation with `tcp.stream eq 131` to inspect it in detail.
4. In the packet Info column (frame 754), found a reference to `upload.file-anon.com` — a file-upload-style domain name. The name alone does not establish the service function or reputation.
5. Classified the session as **suspicious outbound transfer consistent with exfiltration**: an internal host sending a large volume of encrypted data outbound to an external file-upload-style host on port 443.

## Queries

See `queries.txt`.

## Findings

- **Reported in the original analysis:** a large-volume outbound TCP session (`tcp.stream 131`) from an internal host to `203.0.113.99` (`upload.file-anon.com`) on port 443.
- **Observed:** the recorded destination name suggests a file-upload service. Destination approval, actual service function, and reputation were not established.
- **Not retained in the published analysis:** the exact byte count. Encryption does not prevent measuring traffic volume.
- **Cannot be determined from the published analysis:** the specific files transferred, malicious intent, destination approval, or real-world reputation. Encrypted traffic did not expose file contents.
- **Lab indicator:** `203.0.113.99` is a documentation address used in the training capture, not a live threat indicator. The source PCAP and screenshots are not published.

## Conclusion

The evidence supports a **suspicious** large-volume outbound transfer consistent with data exfiltration to an external destination whose approval status is unknown. It does not by itself confirm malicious intent — that distinction would require additional context such as asset ownership, business justification, or DLP/proxy logs for the destination.

## Tier 1 Decision

**Escalate for investigation of a suspected unauthorized outbound transfer.** The capture analysis supports an anomalous outbound session, but does not establish malicious intent or data sensitivity. Request source-host ownership, business justification, and relevant endpoint, proxy, or DLP evidence before final classification. The handover should include the source host when verified, destination IP/domain, port, capture window, and `tcp.stream 131` reference. No numeric severity is assigned without a defined severity matrix and asset context.

## Skills Demonstrated

- Statistics-first triage using conversations and byte volume to identify outliers
- Isolating a specific TCP stream for focused inspection
- Extracting network indicators from packet data
- Distinguishing what a capture can and cannot confirm when traffic is encrypted
- Severity assessment and escalation handoff

## MITRE ATT&CK

**T1567 — Exfiltration Over Web Service**, as a provisional mapping if the transfer is confirmed to use a web service for unauthorized data removal. Port 443 and a file-upload-style name do not alone confirm this technique.
