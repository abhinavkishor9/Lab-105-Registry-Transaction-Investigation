# Lab 105 — Registry Transaction Investigation Timeline

## Overview

This timeline records the known sequence of the Registry investigation on `DESKTOP-9MMM37V` on 13 October 2026.

The baseline timestamp and displayed Sysmon event times are taken from the supplied PowerShell output. Where the exact time of an action was not displayed, the timeline identifies the action without inventing a timestamp.

## Timeline

| Time (IST, UTC+05:30) | Event | Evidence | Assessment |
|---|---|---|---|
| 06:28:31.3738493 | Baseline Registry check | `registry-baseline.json` and PowerShell output | The test key was reported absent before the controlled changes. |
| Not captured in the supplied output | Create Registry key and initial value | `registry-actions.csv` and PowerShell output | PowerShell displayed `LabStatus : Initial`. |
| Not captured in the supplied output | Modify Registry value | `registry-actions.csv` and PowerShell output | PowerShell displayed `LabStatus : Modified`. |
| Not captured in the supplied output | Export Registry key | `registry-after-modification.reg` | The export command reported successful completion. |
| Not captured in the supplied output | Capture current Registry state as JSON | `registry-current-state.json` | The command produced a serialization-depth warning; completeness requires validation. |
| 06:24:45 | Sysmon Registry value-set event | Local Sysmon output, record ID not shown in the supplied excerpt | The excerpt is incomplete, and its relationship to the test key is not established. |
| 06:29:34 | Sysmon Event ID 13 | Local Sysmon output, record ID `1270661` | The displayed event concerned a McAfee Registry value, not the test key. |
| Not captured in the supplied output | Wazuh Registry event reviewed | Wazuh event record ID `1245054`, dated 12 October 2026 | The event concerned McAfee-related Registry activity on an earlier date. |
| Not captured in the supplied output | Transaction assessment recorded | `transaction-assessment.txt` | The notes correctly state that ordinary Registry changes do not prove transactional behavior. |
| Not captured in the supplied output | Cleanup command executed | `registry-cleanup.csv` | Inspect `KeyExistsAfterCleanup` to verify the recorded final state. |
| After evidence collection; exact time not captured | SHA-256 manifest generated | `evidence-hashes.csv` | Hashes were recorded for the nine other evidence files. |

## Timeline Interpretation

The baseline output establishes the initial absence of the test key. The subsequent PowerShell outputs establish that the test value was created with `Initial` and later displayed as `Modified`.

The displayed Sysmon and Wazuh events concern McAfee Registry activity. Their timestamps and target objects do not establish a direct correlation with the controlled Registry test.

The exact creation and modification timestamps should be recovered from `registry-actions.csv` so the operations can be placed accurately in chronological order.

## Evidence Integrity

The generated `evidence-hashes.csv` contains SHA-256 hashes for the other evidence files present at the time of hashing. The manifest excludes itself.

If any evidence file is corrected or regenerated, such as `registry-current-state.json`, the manifest should be regenerated. Preserve earlier manifests separately if maintaining historical integrity records.

## Final Assessment

The available evidence supports a controlled Registry state change and associated evidence collection. The displayed telemetry does not establish direct Sysmon/Wazuh correlation with the test key, and no transaction-specific commit or rollback has been demonstrated.
