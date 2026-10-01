# Detection Logic Review — Splunk SPL

## Scenario

Three training scenarios were reviewed for detection-noise suppression and the risks introduced by exclusions: backup egress, service-account lockouts, and process creation during patch deployment.

## Validation Status

This repository contains SPL search fragments and tuning proposals, not fully validated production detection rules. No before/after alert counts, measured false-positive reduction, or test results are published. The training index is `index=alon`.

## 1. Egress Volume

The recorded search sums `bytes_out` by `src_host`, excluding hours 01:00–02:59, then selects totals over 10,000,000,000 bytes (10 GB in decimal units).

**Limitations:** the total covers the selected search range, not a fixed per-hour window; the query excludes those hours for every host, not only SRV-BKP-01. The hour calculation depends on search time-zone handling. The source field's meaning and whether it measures the intended outbound volume must be verified.

**Coverage risk:** malicious transfers during the excluded hours are also excluded. No false-positive reduction has been measured.

## 2. Lockout Storm

The proposal is to correlate account lockouts with recent help-desk tickets before deciding whether an alert represents expected activity.

**Status:** conceptual logic only; no runnable SPL or ticket schema is supplied.

**Risk:** a ticket is context, not proof of benign activity. Missing, stale, or compromised ticket data can make suppression unsafe. Validate the ticket's timing, scope, approval, and relationship to the affected account.

## 3. Mass Process Execution

The recorded fragment selects EventCode 4688, excludes one exact `cmdline` value, and bins time into ten-minute windows.

**Limitations:** it does not count events, group counts by a verified host/account field, or apply a threshold. It has no SYSTEM-context condition. Therefore it cannot be described as a complete mass-execution detection or a validated SYSTEM-specific exclusion.

**Coverage risk:** exact command-line matching may miss legitimate variants or suppress attacker activity using the same text. Broadly excluding SYSTEM would create a larger blind spot, but that broad exclusion is not implemented here.

## Tier 1 Output

Submit the proposed logic, field requirements, and coverage risks to the detection owner. Deployment and exclusions require the organisation's review and change-control process.

## Completion Criteria

Verify field names and semantics, time range and time zone; complete aggregation and thresholds; test expected benign and suspicious cases; record resulting counts and known blind spots. None of these checks is claimed complete without evidence.

## Published Work

- [Recorded search fragments and conceptual logic](queries.txt)
- [Interview notes](INTERVIEW_NOTES.md)

## Skills Demonstrated

SPL logic review, identification of incomplete aggregation, review of exclusion scope, dependency-risk analysis, and documentation for detection-owner review.

## MITRE ATT&CK

Not mapped: this is a logic-review exercise, not an observed attack.
