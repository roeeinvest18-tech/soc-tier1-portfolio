# Interview Notes — SIEM Detection Rule Tuning

**What happened?**
I worked through several detection-tuning scenarios. For each one, I identified the field that should separate expected from suspicious activity, wrote the SPL logic, and then checked what the rule could still miss.

**What was the initial alert?**
Not applicable — this was a detection-tuning exercise rather than an incident investigation.

**What did I check first?**
For each scenario, I first asked what information actually separates the normal case from the suspicious case. For the egress rule, that was the maintenance window. For the mass-execution rule, it was the known-good SYSTEM deployment command line.

**What evidence mattered most?**
The trade-off created by each tuning decision. Reducing noise can also reduce visibility, so I documented what the rule could miss after each change.

**What did I rule out?**
A broad SYSTEM-context exclusion for the mass-execution rule. It would reduce the noise, but it could also hide attacker activity running as SYSTEM.

**What was my conclusion?**
Two rules (egress volume, mass execution) have a documented false-negative risk. The lockout rule has a dependency risk because its logic relies on an external ticketing system.

**Would I escalate?**
Not applicable as an incident decision. The tuning output would be reviewed by whoever owns the detection content before any exclusions are deployed.

**Why?**
The reviewer needs to understand both what the tuning improves and what visibility it gives up.

**Questions I would expect in an interview:**
- *"Isn't excluding the maintenance window risky?"* — Yes. It reduces the backup-related noise, but an attacker who exfiltrates during that window could be missed by this rule. I would want additional detection coverage for that period.
- *"Why not exclude all SYSTEM process creation?"* — Because that would be too broad. A narrower exclusion for the specific known-good deployment keeps more visibility while still addressing the original noise.
