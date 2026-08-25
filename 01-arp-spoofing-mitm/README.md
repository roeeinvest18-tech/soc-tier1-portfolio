# ARP Spoofing / MITM Detection — Packet Capture Analysis

## Scenario

A packet capture from an internal network was reviewed for signs of ARP cache poisoning. No alert or ticket triggered this review — it was a self-directed packet analysis exercise using Wireshark against a captured ARP conversation on a small local subnet (gateway `192.168.1.1`).

## Initial Evidence

Filtering on ARP reply traffic (`arp.opcode == 2`) surfaced several packets flagged with an **IP duplicate** warning — the first indicator that more than one device on the network was claiming the same IP address.

## Investigation

Alert → identify conflicting claim → map victim scope → confirm impersonation → identify attack cadence → validate the finding:

1. **Confirm the conflict.** `arp.opcode == 2` showed IP-duplicate warnings involving `192.168.1.1` (the default gateway address).
2. **Map the victim scope.** Filtering on the suspect source MAC (`eth.dst == aa:bb:cc:00:00:20`) enumerated every host that had received ARP traffic from it, establishing how many endpoints were touched by the activity.
3. **Confirm impersonation.** `arp.opcode == 2 and arp.src.proto_ipv4 == 192.168.1.1` returned **two different MAC addresses** both claiming to own `192.168.1.1` — one legitimate gateway MAC and one impersonator.
4. **Establish the pattern.** The impersonating MAC was sending replies at a steady ~5-second cadence — a repeating, automated injection rather than a one-off collision.
5. **Cross-reference request vs. reply.** Filtering on genuine ARP requests (`arp.opcode == 1 and arp.dst.proto_ipv4 == 192.168.1.1`) and chronologically matching them against replies distinguished two populations:
   - Endpoints receiving a fast, single, direct reply from the real gateway (**unicast reply**, expected behavior).
   - Dozens of replies arriving from the foreign MAC with **no corresponding request** from the endpoint that received them (**unsolicited ARP replies**) — the signature of active ARP poisoning, not a race condition.

**Process note:** During the original analysis, I identified and corrected several technical interpretation errors before finalizing the findings. The final analysis distinguishes ARP requests from replies, treats the endpoints as independent gateway clients, and correctly identifies the forged gateway address in the ARP source fields.

## Queries

See `queries.txt` for the exact Wireshark display filters used, in the order applied above.

## Findings

- **Confirmed:** two MAC addresses answering for the same gateway IP (`192.168.1.1`) — a classic ARP spoofing signature.
- **Confirmed:** the impersonating MAC sent unsolicited ARP replies (no matching request) to multiple endpoints at a repeating ~5-second interval.
- **Confirmed:** legitimate gateway traffic (fast unicast replies to genuine requests) was present concurrently, confirming the poisoning was injected alongside — not instead of — normal ARP traffic.
- **Cannot be determined from this capture alone:** the identity/location of the physical device behind the impersonating MAC, or whether any credentials/traffic were actually intercepted downstream of the poisoning.

## Conclusion

The evidence supports an active ARP cache poisoning / man-in-the-middle attempt against the gateway address on this subnet. This is a network-layer finding from packet data only — it confirms the poisoning mechanism, not any specific downstream impact (e.g., credential theft), which is outside what this capture can show.

## Tier 1 Decision

**Escalate.** Confirmed ARP spoofing against a default gateway address affecting multiple endpoints is not something a Tier 1 analyst closes or sits on — it's a live MITM condition with data-interception potential. Escalation would include: the impersonating MAC address, the list of affected endpoints (from the victim-scope filter), and the observed 5-second replay cadence as supporting evidence for the network/IR team to locate and isolate the source device.

## Skills Demonstrated

- Constructing and chaining Wireshark display filters to isolate ARP traffic
- Distinguishing legitimate unicast ARP replies from unsolicited ones
- Recognizing repeating unsolicited ARP replies as a poisoning signature
- Validating and correcting technical interpretations before reporting
- Escalation judgment for a confirmed layer-2 MITM condition

## MITRE ATT&CK

**T1557.002 — Adversary-in-the-Middle: ARP Cache Poisoning.** The observed behavior (unsolicited, repeating ARP replies impersonating the default gateway MAC) directly matches this technique's definition.
