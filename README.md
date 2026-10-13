# Lab 105 — Registry Transaction Investigation

## Repository Description

Investigate controlled Windows Registry modifications using PowerShell, Sysmon, and Wazuh. Examine Registry telemetry, validate evidence collection, document limitations in transaction attribution, and preserve investigation artifacts with SHA-256 hashes.

## Overview

Windows Registry activity is important during endpoint investigations because Registry keys and values can be modified by applications, services, installers, administrative scripts, and potentially malicious processes.

This lab establishes a controlled Registry test key, records an initial value, modifies that value, collects available Sysmon Registry events, and preserves the resulting evidence for investigation.

The exercise also examines an important forensic distinction: observing an ordinary Registry modification does not establish that a transactional Registry operation was performed.

## Lab Objectives

- Establish a controlled Registry test environment.
- Record the Registry key's baseline state.
- Create a Registry key and an initial string value.
- Modify the value and record the corresponding action in an evidence log.
- Export the Registry key and capture its current state.
- Collect Sysmon Registry events and investigate their relevance.
- Search Wazuh for Registry-related telemetry.
- Assess the limitations of the available evidence.
- Clean up the test key and generate SHA-256 hashes for evidence files.

## Lab Environment

| Component | Configuration |
|---|---|
| Operating system | Windows 11 Pro |
| Host | `DESKTOP-9MMM37V` |
| Shell | PowerShell 7 |
| Registry location | `HKCU:\Software\RegistryTransactionLab` |
| Evidence directory | `C:\RegistryTransactionLab\Evidence` |
| Endpoint telemetry | Sysmon |
| SIEM | Wazuh |
| Wazuh agent | `001` |

The Sysmon and Wazuh services were both running when the lab environment was checked.

## Investigation Scenario

A controlled Registry modification is performed on a Windows endpoint to evaluate how Registry changes can be documented and investigated.

The test creates the following Registry value:

- **Key:** `HKCU:\Software\RegistryTransactionLab`
- **Value name:** `LabStatus`
- **Initial value:** `Initial`
- **Modified value:** `Modified`

The investigation records the expected actions, exports the Registry state, and reviews available Sysmon and Wazuh events.

The central question is whether the collected evidence demonstrates the controlled Registry changes and whether it establishes any transaction-specific behavior.

## Investigation Workflow

1. Initialize the evidence directory and record system information.
2. Confirm the initial state of the test Registry key.
3. Create the key and set `LabStatus` to `Initial`.
4. Change `LabStatus` to `Modified`.
5. Record the operations in `registry-actions.csv`.
6. Export the Registry key and current state.
7. Collect Sysmon Registry events with event IDs 12, 13, and 14.
8. Search Wazuh for relevant Registry events and correlate available fields.
9. Assess the evidence and document unresolved questions.
10. Remove the test key and record the cleanup result.
11. Generate a SHA-256 evidence manifest.

## Evidence Collected

The evidence directory contains the following artifacts:

| Artifact | Purpose |
|---|---|
| `investigation-start.txt` | Records the investigation start time |
| `system-information.txt` | Records selected endpoint details |
| `registry-baseline.json` | Documents the baseline Registry state |
| `registry-actions.csv` | Records the controlled Registry operations |
| `registry-after-modification.reg` | Exports the Registry key |
| `registry-current-state.json` | Captures the current Registry state |
| `sysmon-registry-events.csv` | Stores collected Sysmon Registry events |
| `transaction-assessment.txt` | Documents the transaction-related conclusion |
| `registry-cleanup.csv` | Records the cleanup action and resulting key state |
| `evidence-hashes.csv` | Stores SHA-256 hashes of the other evidence files |

## Findings

- The baseline output showed that the test key did not exist before the controlled exercise.
- PowerShell output confirmed that the `LabStatus` value was initially `Initial`.
- Subsequent PowerShell output confirmed that the value was changed to `Modified`.
- The Registry export completed successfully.
- The JSON serialization produced a warning that the output was truncated because the serialization depth exceeded the default depth of 2.
- The local Sysmon query returned Registry value-set events, including McAfee-related activity. The displayed events did not identify the controlled test key.
- A Wazuh event inspected during the investigation also showed unrelated Registry activity. It was not evidence of the controlled test operation.
- The evidence-hash command generated a SHA-256 manifest for the nine other files in the evidence directory.

The available screenshots do not establish that a matching Sysmon or Wazuh event for the test key was collected. The contents of the complete exported CSV files should be examined before making a definitive statement about whether matching telemetry exists.

## Limitations

This lab demonstrates controlled Registry modifications and the collection of potentially relevant telemetry. It does not, by itself, demonstrate a genuine transactional Registry operation.

A normal PowerShell operation such as `New-ItemProperty` or `Set-ItemProperty` changes Registry state but does not prove that a transaction was used, committed, or rolled back.

Additional limitations include:

- The displayed Sysmon events concern unrelated Registry activity.
- The available screenshots do not establish successful Wazuh ingestion of the test events.
- The current-state JSON export reported a serialization-depth warning.
- A SHA-256 manifest records file hashes but does not independently establish evidence authenticity, completeness, or chain of custody.
- The cleanup command was executed, but its recorded result should be inspected to confirm the final key state.

## Conclusion

The lab successfully demonstrated a controlled Registry value change from `Initial` to `Modified`, created supporting evidence artifacts, and generated a SHA-256 manifest.

However, the supplied telemetry does not yet establish a direct correlation between the controlled Registry operations and matching Sysmon or Wazuh events. Transaction-specific behavior also remains unproven.

The next investigative step is to validate the complete event export, correct the JSON serialization warning, and determine whether the expected Registry activity is present in the endpoint and SIEM telemetry.

## Key Learning

**A Registry state change, a Registry telemetry event, and a transactional Registry operation are three different things.** A sound investigation distinguishes what was performed, what was observed, and what the available evidence can actually prove.
