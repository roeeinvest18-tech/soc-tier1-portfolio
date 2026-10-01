# Interview Notes — ARP Spoofing / MITM Indicators

**What triggered the review?**
A self-directed training-capture review, not a SIEM alert.

**What did I check?**
ARP replies, duplicate gateway-IP claims, and request/reply timing. The original analysis describes unsolicited replies at roughly five-second intervals.

**What is the assessment?**
Suspected ARP cache poisoning. The pattern warrants escalation, but does not prove interception or credential theft.

**What needs correction?**
The recorded scope filter uses eth.dst while its explanation described source traffic. I would verify the suspect MAC in the PCAP, filter eth.src, and examine recipients before claiming a victim count.

**What alternatives remain?**
Misconfiguration, legitimate gateway redundancy, or another source of conflicting ARP claims. Cadence alone does not rule them out.

**What would I hand over?**
Capture time window, verified conflicting MAC/IP claims, example packet pairs, affected endpoints if verified, and a request for network/IR validation under the approved workflow.

**What can the published work prove?**
It shows my documented method and the review of its limitations. The original PCAP is not included, so readers cannot independently reproduce the finding.
