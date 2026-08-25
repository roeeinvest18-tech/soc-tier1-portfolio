# Interview Notes — Network Exfiltration Triage

**What happened?**
A capture with several unrelated suspicious actors was escalated for triage with no further context. I worked one suspect through to a documented finding: a large outbound transfer to an external host.

**What was the initial alert?**
A one-line escalation note: "Multiple suspicious activities. Triage." No IOCs or direction were provided, so I had to narrow down and classify the traffic myself.

**What did I check first?**
Statistics → Conversations → TCP sorted by bytes. With no starting IOC, this quickly showed which conversations were outliers before I looked at individual packets.

**What evidence mattered most?**
The byte-volume outlier and the destination domain name (`upload.file-anon.com`) visible in the stream metadata. The domain name pattern was consistent with an external file-upload service and was unusual for routine business traffic.

**What did I rule out?**
I did not rule out legitimate-but-unsanctioned use. The capture alone could not distinguish an employee using a personal file-sharing site from malicious exfiltration, so I recorded that as "cannot be determined" rather than assuming intent.

**What was my conclusion?**
A suspicious outbound transfer consistent with exfiltration: large volume, encrypted traffic, and an external file-upload-style destination. The capture did not confirm malicious intent by itself.

**Would I escalate?**
Yes, severity 4/5 in the exercise's rating scheme. It was worth prompt follow-up, but the capture alone did not show enough to call it a critical incident.

**Why?**
The destination and transfer volume were enough to require more context, such as asset ownership, business justification, and DLP or proxy logs.

**Questions I would expect in an interview:**
- *"Why sort by bytes instead of reading packets first?"* — With an unscoped capture and no starting IOC, sorting conversations by volume is a quick way to find outliers. I can then focus packet-level analysis on the most relevant streams.
- *"How would you check whether this was legitimate traffic?"* — I would check DLP/proxy logs, whether the destination is approved, and the asset owner's business justification. None of that was available in the capture itself.
