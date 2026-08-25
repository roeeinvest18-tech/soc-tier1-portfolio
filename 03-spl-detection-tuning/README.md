# SIEM Detection Rule Tuning — Splunk SPL

## Scenario

Three noisy or gap-prone detection ideas were reviewed and tuned in Splunk against a training dataset (`index=alon`). For each, a distinguishing field was identified, an SPL rule was written to operationalize it, and the rule was then checked for blind spots.

## Initial Evidence

Each of the three scenarios below started from a specific noise source behind the alert: a scheduled backup job on host `SRV-BKP-01` tripping an egress-volume alert, a service account's lockouts flooding a queue meant for real lockout storms, and a known patch-deployment tool (`ccmexec.exe`) tripping a mass-execution alert. In each case, the fix required trading a known false positive for a named, documented false negative or dependency risk rather than eliminating noise for free.

## Investigation

For each scenario: identify what should distinguish malicious from benign → write the SPL rule around that field → check what activity the rule would still miss.

### 1. Egress volume — SRV-BKP-01

- **Distinguishing field:** whether the large outbound transfer is legitimate (e.g., a scheduled backup) or not.
- **Rule:** see `queries.txt` #1. Sums outbound bytes per host over all hours *except* the 01:00-03:00 maintenance window, and alerts above a 10GB threshold.
- **Blind spot:** excluding the maintenance window from the volume calculation also blinds the rule to an attacker who times their exfiltration to happen *during* that same window — the exclusion reduces false positives from legitimate backups, but at the cost of a matching false negative.

### 2. Lockout storm — svc_sql

- **Distinguishing field:** whether a matching help-desk ticket exists for the account lockout.
- **Rule:** look up recent help-desk tickets for the same user before alerting on the lockout storm.
- **Blind spot:** this makes the detection's reliability dependent on an external system (the ticketing platform) — if the ticketing system is slow, incomplete, or itself compromised, the suppression logic is only as good as that dependency.

### 3. Mass execution — ccmexec

- **Distinguishing field:** an expected SYSTEM-context deployment (`ccmexec.exe /deploy KB5041585`) that should be excluded from a mass-process-execution alert.
- **Rule:** see `queries.txt` #3. Filters process creation events (`EventCode 4688`) while excluding that specific known-good SYSTEM command line, then bins the events into 10-minute windows.
- **Blind spot:** excluding SYSTEM-run activity broadly (to suppress this one known-good deployment) would allow any attacker activity that also runs as SYSTEM to pass this rule.

## Queries

See `queries.txt` for the three SPL rules as written.

## Findings

- Two tuned SPL rules (egress volume, mass execution) have a documented false-negative trade-off introduced by their tuning logic.
- One rule (lockout storm) has a dependency risk involving the external ticketing system rather than a direct coverage gap.
- Each exclusion added to reduce noise was paired with a note describing what it could miss.

## Conclusion

This is detection-rule tuning rather than an incident investigation. The main result is that each tuning change reduced a specific source of noise while introducing a documented coverage or dependency risk.

## Tier 1 Decision

Not applicable as an incident close/escalate decision. This output would be reviewed by whoever owns the detection content before any exclusions are deployed. The documented blind spots are the main trade-offs that reviewer would need to consider.

## Skills Demonstrated

- Writing SPL to operationalize a detection idea (`stats`, `eval`, `bin`, time-window logic, field exclusions)
- Identifying the distinguishing field between benign and suspicious activity before writing the rule
- Reviewing detection logic for the false-negative it introduces, not just the false positives it suppresses
- Recognizing when a detection depends on an external system such as a help-desk ticketing platform
- Documenting tuning trade-offs clearly for review

## MITRE ATT&CK

Not mapped. This project is about detection-rule tuning and trade-offs, not an observed attack technique.
