# Incident investigation & fix templates (team-dispatch companion)

Reference formats for using team-dispatch in the flow where the user **brings the code themselves** to confirm and fix an incident. git clone/push/PR are never automated (the user does them). This file layers incident context onto the contract freeze (backbone step 2) and the integration report (backbone step 5).

Recommended mapping: confirming the problem = **mode D** (bug investigation); fixing = single owner per file; review = **mode B** (review fan-out).

---

## Incident contract template (freeze before spawning owners)

```
incident_id:
repo / branch / commit:        # where the user's checked-out code lives (informational)

reported_symptom:              # the symptom
expected_behavior:             # what should happen
failing_evidence:              # logs, stack traces, failing tests
reproduction_command:          # command/steps to reproduce

affected_surface:              # blast radius (modules, APIs, screens)
non_goals:                     # what this incident will NOT touch
allowed_files:                 # files that may be modified
file_owners:                   # single write owner per file
verification_gates:            # commands that decide "done" (test/lint/typecheck/original repro)
rollback_or_mitigation_notes:  # rollback and mitigation options
```

## Final report template (lead → human — maps to the four-part report)

```
## Summary
## Root cause
## Changes made
## Files changed/created       # confirm this matches file_owners
## Verification results        # integration gates the lead re-ran itself in the working tree (pass/fail per command)
## Remaining risks / follow-up
## Next actions                # commit/push/PR are the user's — list only the commands to run next
```
