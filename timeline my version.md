# Lab 105 — Registry Transaction Investigation Timeline

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

