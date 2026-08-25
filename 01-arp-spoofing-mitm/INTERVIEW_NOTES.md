# Interview Notes — ARP Spoofing / MITM Detection

**What happened?**
While reviewing a packet capture from a small internal network, I noticed
ARP replies flagged with an IP-duplicate warning for the gateway address.

**What was the initial alert?**
There wasn't one — this was a self-directed capture review, not a triggered
SIEM/IDS alert. I found the anomaly by filtering ARP traffic directly.

**What did I check first?**
`arp.opcode == 2` to see all ARP replies and confirm the duplicate-IP
warning was real and not a one-off glitch.

**What evidence mattered most?**
Two different MAC addresses both answering for `192.168.1.1`, and — more
importantly — dozens of ARP replies with no matching prior request
(unsolicited replies). A single duplicate-IP warning could be a
misconfiguration; a pattern of unsolicited replies at a fixed ~5-second
interval is not.

**What did I rule out?**
A simple IP conflict from misconfiguration (e.g., two devices manually
assigned the same static IP by mistake). That would produce occasional
duplicate warnings but not a steady stream of unsolicited replies impersonating
the gateway specifically.

**What was my conclusion?**
Active ARP cache poisoning against the default gateway, confirmed at the
packet level. I could not confirm downstream impact (e.g., whether traffic
was actually intercepted or credentials captured) from this capture alone.

**Would I escalate?**
Yes. A confirmed MITM condition against a gateway address affecting
multiple hosts is outside Tier 1 scope to resolve — it needs the
network/IR team to physically locate and isolate the source.

**Why?**
Because ARP poisoning at the gateway can expose all traffic from the
affected hosts to interception, and the attacker's device needs to be
located on the physical/switch layer, which is beyond log/packet analysis
alone.

**Question an interviewer might ask, and how I'd answer:**
- *"How do you tell a real ARP conflict from a misconfiguration?"* — Look for
  a pattern: repeating, unsolicited replies (no matching request) from one
  MAC impersonating another device's IP, versus a single sporadic conflict.
- *"What would you have done differently if you had live network access
  instead of just a capture?"* — Cross-reference the impersonating MAC
  against the switch's CAM table / DHCP lease table to physically locate the
  device, and consider Dynamic ARP Inspection (DAI) as a longer-term
  mitigation — though I'd flag that as a recommendation, not something I
  implemented in this exercise.
