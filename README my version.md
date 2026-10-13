# Lab-105-Registry-Transaction-Investigation
## Overview
The Windows Registry is a hierarchical database used to store configuration settings for Windows, applications, services, and user profiles.
Its main components include:
- Keys: Containers that organize Registry settings.
- Values: Named settings stored within keys.
- Data: The information assigned to a value, such as text or a number.
- Hives: Major Registry data structures, such as HKEY_CURRENT_USER and HKEY_LOCAL_MACHINE.

  A Registry transaction is a transactional operation in which Registry changes can be committed or rolled back as a unit. Windows supports transactional Registry operations through mechanisms such as Kernel Transaction Manager (KTM) and transactional Registry APIs, although these mechanisms are not the normal way most applications modify Registry values.
A transaction-aware investigation may need evidence of:
- Transaction creation or association.
- Registry operations performed within the transaction.
- Whether changes were committed or rolled back.
- The process responsible for the activity.
- Relevant timestamps and supporting system artifacts.
A before-and-after Registry comparison can demonstrate that a value changed, but it does not prove that the change was transactional.

Windows Registry activity is important during endpoint investigations because Registry keys and values can be modified by applications, services, installers, administrative scripts, and potentially malicious processes.

This lab establishes a controlled Registry test key, records an initial value, modifies that value, collects available Sysmon Registry events, and preserves the resulting evidence for investigation.

The exercise also examines an important forensic distinction: observing an ordinary Registry modification does not establish that a transactional Registry operation was performed.

## Lab Objectives
The objective of this lab is to investigate controlled Windows Registry modifications, examine available endpoint telemetry, and evaluate the limitations of using Registry events to establish transactional behavior.
- **Establish a Registry baseline:** Verify whether the designated test key exists before the investigation and document its initial state with a timestamp.
- **Perform controlled Registry modifications:** Create a dedicated test key, set an initial Registry value, and modify it while recording each operation in an evidence log.
- **Preserve Registry artifacts:** Export the test key and capture its current state to support subsequent analysis and validation.
- **Investigate Sysmon Registry telemetry:** Examine Registry event IDs 12, 13, and 14 to identify key creation or deletion, value modifications, and key or value renaming.
- **Correlate endpoint and SIEM evidence:** Search Wazuh for Registry-related events and compare available timestamps, target objects, process identifiers, executable paths, and value data against the controlled test activity.
- **Differentiate relevant events from unrelated activity:** Evaluate whether collected Sysmon and Wazuh events reference the test Registry key rather than unrelated system or application activity.
- **Assess transactional evidence:** Distinguish ordinary Registry modifications from transaction-specific operations and identify what additional evidence would be required to establish transaction usage, commit, or rollback.
- **Validate evidence collection:** Inspect exported files, identify incomplete or truncated output, and verify that the recorded artifacts accurately represent the investigation.
- **Maintain evidence integrity:** Generate SHA-256 hashes for collected artifacts and regenerate the manifest whenever evidence files are modified.
- **Document cleanup and investigation limitations:** Record the test-key cleanup result, identify telemetry gaps, and clearly separate confirmed observations from unverified conclusions.


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

## Lab Scenario
A SOC analyst is investigating Registry activity on a Windows endpoint to understand how controlled Registry modifications can be observed, recorded, and correlated with endpoint security telemetry. Registry changes may occur during legitimate application activity, system configuration, software installation, or potentially malicious behavior. However, observing a Registry change alone does not establish malicious intent or prove that a transactional Registry operation occurred.
The investigation uses a controlled test key, `HKCU\Software\RegistryTransactionLab`, to simulate a sequence of Registry modifications without altering unrelated system configuration. The analyst first establishes a baseline, creates the test key with a `LabStatus` value set to `Initial`, and then changes the value to `Modified`. Each operation is documented to provide a reference for examining the collected evidence.
The analyst then exports the Registry key, records its current state, and examines Sysmon Registry events with event IDs 12, 13, and 14. The corresponding Wazuh telemetry is reviewed to determine whether the controlled activity can be identified and correlated with the expected Registry path, timestamps, process details, and value data.
During the investigation, the available Sysmon and Wazuh results include Registry activity associated with McAfee components. These events provide examples of Registry telemetry but do not directly establish that the controlled test key was modified by the observed processes. The analyst must distinguish unrelated system activity from evidence relevant to the test instead of assuming that every Registry event belongs to the investigation.
The evidence collection also presents validation challenges. The JSON state export reports a serialization-depth warning, requiring the analyst to inspect and correct the output before treating it as a complete artifact. The cleanup result and SHA-256 manifest must also be reviewed to confirm the recorded state and support evidence integrity.
### Investigation Questions
- Was the test Registry key absent before the controlled exercise?
- Can the initial and modified `LabStatus` values be established from the collected evidence?
- Do the available Sysmon and Wazuh events directly reference the test Registry key?
- Which event fields can help correlate Registry activity with a process and a specific time?
- What conclusions are justified when expected telemetry is not found in the examined results?
- Does the evidence demonstrate an actual transactional Registry operation, commit, or rollback?
### Expected Outcome
The analyst should produce a documented sequence of controlled Registry changes, evaluate the relevance and completeness of the collected telemetry, preserve evidence with SHA-256 hashes, and record any unresolved gaps. The final assessment must distinguish confirmed Registry modifications from unverified telemetry correlations and transaction-specific behavior.

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

