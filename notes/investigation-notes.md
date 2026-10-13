# Lab 105 — Registry Transaction Investigation Notes

## 1. Investigation Summary

This investigation examined controlled Registry modifications on the Windows endpoint `DESKTOP-9MMM37V`.

The objective was to establish a baseline, create and modify a test Registry value, collect available Sysmon events, review Wazuh telemetry, preserve supporting artifacts, and evaluate whether the evidence demonstrated transactional Registry behavior.

The investigation used the following Registry location:

`HKCU:\Software\RegistryTransactionLab`

The value under examination was `LabStatus`.

## 2. Baseline and Controlled Changes

### Baseline

The baseline output recorded the following:

| Field | Observed value |
|---|---|
| Registry path | `HKCU:\Software\RegistryTransactionLab` |
| Key existed before the test | `False` |
| Baseline timestamp | `2026-10-13T06:28:31.3738493+05:30` |

This establishes that the test key was reported absent at the time of the baseline check.

### Registry Creation

PowerShell created the test key and added a string value:

| Field | Value |
|---|---|
| Value name | `LabStatus` |
| Value data | `Initial` |
| Operation | Create key and set initial value |

The subsequent PowerShell output displayed `LabStatus : Initial`.

### Registry Modification

The test value was subsequently changed to `Modified`.

| Field | Value |
|---|---|
| Value name | `LabStatus` |
| Previous value | `Initial` |
| New value | `Modified` |
| Operation | Modify existing Registry value |

The PowerShell output displayed `LabStatus : Modified`, confirming the resulting value at the time of that check.

The corresponding action records were written to `registry-actions.csv`. The precise timestamps of the creation and modification operations should be taken from that CSV rather than inferred from screenshot order.

## 3. Registry Export and State Capture

The following export command completed successfully:

`reg.exe export "HKCU\Software\RegistryTransactionLab" "C:\RegistryTransactionLab\Evidence\registry-after-modification.reg" /y`

The command returned:

`The operation completed successfully.`

A separate JSON state capture produced the warning:

`Resulting JSON is truncated as serialization has exceeded the set depth of 2.`

This warning means the JSON output should not be accepted as a complete state capture without further validation.

For a simple Registry key, a corrected capture can use an explicit Registry provider path and a greater serialization depth:

```powershell
$LabPath = "C:\RegistryTransactionLab\Evidence"
$RegPath = "HKCU:\Software\RegistryTransactionLab"

Get-ItemProperty -LiteralPath $RegPath |
    ConvertTo-Json -Depth 5 |
    Out-File (Join-Path $LabPath "registry-current-state.json") -Encoding utf8
```

After regenerating the file, inspect its contents and rerun the evidence-hashing process.

## 4. Sysmon Investigation

The local Sysmon query requested event IDs 12, 13, and 14 from the preceding 30 minutes.

| Event ID | General meaning |
|---|---|
| 12 | Registry key/value object creation or deletion |
| 13 | Registry value set |
| 14 | Registry key/value rename |

The collected output included a Registry value-set event with the following attributes:

| Field | Observed value |
|---|---|
| Event ID | 13 |
| Record ID | 1270661 |
| Local event time | `2026-10-13 06:29:34` |
| Process ID | 5516 |
| Image | `C:\Program Files\McAfee\wps\1.40.161.1\mc-fw-host.exe` |
| Target object | `HKLM\SOFTWARE\McAfee\wps\TaskScheduler\NGMCadence` |
| User | `NT AUTHORITY\SYSTEM` |

This event records McAfee-related Registry activity. Its target object is not the lab test key, so it cannot be used as direct evidence of the controlled `LabStatus` modification.

Another displayed Registry event occurred at `2026-10-13 06:24:45`. The supplied excerpt is incomplete, and its relevance to the test key has not been established.

### Assessment

The screenshots demonstrate that Sysmon Registry telemetry was available locally. They do not demonstrate that the displayed events correspond to the test operations.

The complete `sysmon-registry-events.csv` file must be searched for:

- `RegistryTransactionLab`
- `LabStatus`
- `HKCU\Software\RegistryTransactionLab`

A matching event, if present, should be examined for its timestamp, event ID, target object, value data, process image, process ID, and user.

## 5. Wazuh Investigation

A Wazuh event inspected during the investigation showed:

| Field | Observed value |
|---|---|
| Event ID | 13 |
| Record ID | 1245054 |
| UTC event time | `2026-10-12 02:25:54.759` |
| Process image | `C:\WINDOWS\system32\services.exe` |
| Target object | `HKLM\System\CurrentControlSet\Services\McAfee Scheduled Task - (McAfee-Dynamicappdownloader)\ImagePath` |
| User | `NT AUTHORITY\SYSTEM` |

The event concerned McAfee-related Registry activity on an earlier date. It is not direct evidence of the controlled test key.

The Wazuh investigation should use the test key name and value name to locate relevant events. If no matching event is found, the correct conclusion is that matching telemetry was not located in the examined results, not that the Registry operation did not occur.

## 6. Transaction Assessment

The test used standard PowerShell Registry operations to create a key, set a value, and modify that value.

These operations demonstrate ordinary Registry state changes. The available evidence does not show that a transactional Registry API was used or that a transaction was committed or rolled back.

The assessment recorded in `transaction-assessment.txt` correctly identifies this limitation.

A transaction-specific investigation would require additional evidence from a transaction-aware test harness or other reliable transaction-specific artifacts.

## 7. Cleanup and Evidence Preservation

A cleanup command was executed to remove the test key if it existed. The command then recorded the timestamp, requested action, Registry path, and post-cleanup key-existence result in `registry-cleanup.csv`.

The actual CSV contents should be inspected to verify that `KeyExistsAfterCleanup` was `False`.

The SHA-256 manifest contained hashes for nine evidence files, including the baseline, action log, Registry export, Sysmon event export, transaction assessment, cleanup log, and system information.

The hashing command excluded `evidence-hashes.csv` itself. This is expected when creating a manifest that hashes the other files in the same directory.

A hash manifest provides integrity-checking values for later comparison. It does not, by itself, prove that the underlying evidence is authentic or complete.

## 8. Final Assessment

### Confirmed

- The baseline output reported that the test key did not exist before the exercise.
- PowerShell displayed the initial value `Initial`.
- PowerShell subsequently displayed the modified value `Modified`.
- The Registry export command completed successfully.
- Sysmon Registry events were available in the queried time window.
- The displayed Sysmon and Wazuh events involved unrelated McAfee Registry activity.
- A SHA-256 manifest was generated for the evidence files.

### Not established by the supplied screenshots

- A matching Sysmon event for the test Registry key.
- A matching Wazuh event for the test Registry key.
- Complete and untruncated JSON state capture.
- Successful post-cleanup absence of the key, without reviewing the CSV result.
- Transactional Registry usage, commit, or rollback.

### Conclusion

The controlled Registry modification is supported by the PowerShell output and associated evidence artifacts. Direct SIEM/endpoint telemetry correlation and transactional behavior remain unproven in the supplied results.
