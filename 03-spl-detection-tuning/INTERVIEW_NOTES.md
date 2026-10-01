# Interview Notes — Detection Logic Review

**What did I do?**
Reviewed training search fragments and proposals for egress, lockouts, and process creation, focusing on noise suppression and coverage risks.

**What is complete?**
The documented logic review. These are not validated production rules, and I do not claim measured false-positive reduction.

**What did the review identify?**
The egress query excludes hours for all hosts and aggregates over the selected search range. The lockout scenario has no executable SPL. The process fragment bins time but has no event count, threshold, or SYSTEM condition.

**What would I do next?**
Verify the actual fields, search window and time zone; complete the rule logic; test benign and suspicious cases; record output counts and failures; submit for detection-owner review.

**Why not suppress based only on a ticket?**
A ticket is enrichment. It needs to match the account, time, approved activity, and incident context; it does not prove the alert is benign.

**Why not exclude all SYSTEM events?**
That would hide malicious activity running in the same context. Even a narrow command-line exclusion needs validation and review.

**What is the Tier 1 deliverable?**
A review package with the proposed changes, required fields, test evidence when available, and remaining blind spots. Deployment follows approved change control.
