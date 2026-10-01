# ARP Spoofing / MITM Indicators — Packet Capture Review

## Scenario

A self-directed Wireshark review of a training capture from a small local subnet identified conflicting ARP claims for gateway `192.168.1.1`. No SIEM/IDS alert triggered this review.

## Investigation

1. Filtered ARP replies with `arp.opcode == 2` and noted duplicate-IP warnings.
2. Reviewed replies claiming `192.168.1.1` with `arp.opcode == 2 and arp.src.proto_ipv4 == 192.168.1.1`. The original notes describe two different MAC addresses claiming the gateway IP.
3. Compared gateway requests using `arp.opcode == 1 and arp.dst.proto_ipv4 == 192.168.1.1` against replies. The notes describe unsolicited replies at approximately five-second intervals.
4. Reviewed endpoint scope. The originally recorded filter used `eth.dst` while its explanation referred to the suspect source MAC. That mismatch prevents treating it as a validated victim-scope filter.

See [queries.txt](queries.txt) for the recorded filters and the correction that requires source-capture verification.

## Findings and Confidence

- **Reported in the original analysis:** two MAC addresses claiming the gateway IP, concurrent legitimate replies, and repeating unsolicited replies.
- **Assessment:** this pattern is consistent with ARP cache poisoning and warrants investigation. Duplicate-IP warnings or reply cadence alone do not establish malicious intent.
- **Not independently verified from the published artifacts:** the legitimate gateway MAC, suspect source MAC, exact affected-host count, and request/reply pairing.
- **Not established:** physical device identity, successful interception, credential theft, or current activity.

The original PCAP and screenshots are not published. This case documents the analysis method; it does not let a reader reproduce the packet-level finding.

## Tier 1 Decision

**Escalate the suspected gateway-impersonation activity for network/IR validation.** Supply the capture time window, conflicting MAC/IP claims, example request/reply pairs, and affected endpoints once verified. In a live environment, check the gateway configuration and switch/asset records under the approved workflow before attributing the activity or requesting isolation.

## Skills Demonstrated

- Using Wireshark ARP display filters
- Comparing requests and replies to evaluate unsolicited traffic
- Distinguishing reported observations from independently reproducible evidence
- Recognizing a source/destination filter mismatch
- Documenting uncertainty and escalation rationale

## MITRE ATT&CK

**T1557.002 — Adversary-in-the-Middle: ARP Cache Poisoning**, as a behavioral hypothesis consistent with the reported pattern. Successful interception is not established.

## Evidence Needed to Complete Validation

Original PCAP or selected packet excerpts; capture timestamps; verified gateway and suspect MAC addresses; example paired and unsolicited replies; distinct endpoint count. No missing values have been reconstructed.
