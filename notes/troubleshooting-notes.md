# Lab 105 — Registry Transaction Investigation Troubleshooting Notes

## 1. Sysmon Returned Registry Events but Not the Test Key

### Observed behavior

The local Sysmon query returned Registry event IDs 12, 13, and 14. The displayed events involved McAfee Registry activity rather than:

`HKCU\Software\RegistryTransactionLab`

### Likely explanation

The query selected Registry events within the last 30 minutes without filtering for the test key. Therefore, unrelated system and application activity was also returned.

The available screenshots do not establish whether the complete exported CSV contains a matching event.

### Troubleshooting

Search the collected CSV:

```powershell
$LabPath = "C:\RegistryTransactionLab\Evidence"
$EventFile = Join-Path $LabPath "sysmon-registry-events.csv"

Import-Csv -LiteralPath $EventFile |
    Where-Object {
        $_.Message -match "RegistryTransactionLab|LabStatus"
    } |
    Format-List *
```

If no rows are returned, inspect the full file and verify whether Sysmon logged the test operations.

Also confirm that the Sysmon configuration includes the Registry events relevant to the operation. Event IDs 12, 13, and 14 do not guarantee that every Registry action will be captured under every configuration.

Do not interpret unrelated Registry events as proof of the test operation.

## 2. Wazuh Did Not Show a Matching Test Event

### Observed behavior

The displayed Wazuh event was a Registry value-set event involving `services.exe` and a McAfee-related Registry path.

### Troubleshooting

Search Wazuh Discover using:

```text
"RegistryTransactionLab"
```

Then try:

```text
"LabStatus"
```

To review Sysmon Registry events more broadly:

```text
data.win.system.providerName:"Microsoft-Windows-Sysmon" AND data.win.system.eventID:(12 OR 13 OR 14)
```

To inspect event ID 13:

```text
data.win.system.providerName:"Microsoft-Windows-Sysmon" AND data.win.system.eventID:13
```

Check the actual event fields and time range. The field `data.win.system.providerName` may be absent or represented differently in some indexed documents, so inspect a known Sysmon event if the query returns no results.

### Interpretation

If the test key does not appear, establish whether:

- The event was generated locally.
- The event was included in the exported Sysmon results.
- The Wazuh agent collected the relevant event.
- The event reached the Wazuh manager.
- The event was indexed in the expected index and time range.

An empty search result alone does not identify which stage failed.

## 3. JSON Serialization Warning

### Observed behavior

The JSON export displayed:

`Resulting JSON is truncated as serialization has exceeded the set depth of 2.`

### Cause

`ConvertTo-Json` uses a default serialization depth that may be insufficient for nested PowerShell objects.

### Corrected command

```powershell
$LabPath = "C:\RegistryTransactionLab\Evidence"
$RegPath = "HKCU:\Software\RegistryTransactionLab"
$JsonFile = Join-Path $LabPath "registry-current-state.json"

Get-ItemProperty -LiteralPath $RegPath |
    ConvertTo-Json -Depth 5 |
    Out-File -LiteralPath $JsonFile -Encoding utf8

Get-Content -LiteralPath $JsonFile
```

For this simple Registry key, the primary concern is obtaining readable output without a truncation warning. Inspect the resulting JSON rather than assuming that increasing the depth alone guarantees completeness.

After replacing the JSON file, regenerate the SHA-256 manifest.

## 4. Verify the Registry Provider Path

Use the PowerShell Registry provider path:

```powershell
$RegPath = "HKCU:\Software\RegistryTransactionLab"

Test-Path -LiteralPath $RegPath
Get-ItemProperty -LiteralPath $RegPath
```

For `reg.exe`, use the native Registry path:

```powershell
reg.exe query "HKCU\Software\RegistryTransactionLab"
```

The PowerShell provider path and the native `reg.exe` path use different formats. Use the appropriate form for each command.

## 5. Validate the Cleanup Result

### Expected behavior

The cleanup command removes only the lab test key if it exists, then records whether it remains.

Inspect the recorded result:

```powershell
$LabPath = "C:\RegistryTransactionLab\Evidence"
$CleanupFile = Join-Path $LabPath "registry-cleanup.csv"

Import-Csv -LiteralPath $CleanupFile |
    Format-List *
```

You can also check the current state:

```powershell
Test-Path -LiteralPath "HKCU:\Software\RegistryTransactionLab"
```

A result of `False` indicates that the key was absent at the time of the check.

If the key is already absent, do not recreate it merely to make the cleanup record appear successful. Preserve the existing evidence and document the actual state.

## 6. Verify the Evidence Directory Before Hashing

### Potential issue

If `$LabPath` is empty or incorrect, a file-enumeration command may inspect an unintended directory.

### Validation

```powershell
$LabPath = "C:\RegistryTransactionLab\Evidence"

if (-not (Test-Path -LiteralPath $LabPath -PathType Container)) {
    throw "Evidence directory not found: $LabPath"
}

Get-ChildItem -LiteralPath $LabPath -File |
    Select-Object Name, Length, LastWriteTime
```

Confirm that the listed files belong to the investigation before generating the manifest.

## 7. Regenerate the SHA-256 Manifest

After correcting or updating evidence files, regenerate the hashes:

```powershell
$LabPath = "C:\RegistryTransactionLab\Evidence"

if (-not (Test-Path -LiteralPath $LabPath -PathType Container)) {
    throw "Evidence directory not found: $LabPath"
}

$HashFile = Join-Path $LabPath "evidence-hashes.csv"

$Hashes = Get-ChildItem -LiteralPath $LabPath -File |
    Where-Object { $_.FullName -ne $HashFile } |
    ForEach-Object {
        Get-FileHash -LiteralPath $_.FullName -Algorithm SHA256
    } |
    Select-Object Path, Algorithm, Hash

$Hashes |
    Export-Csv -LiteralPath $HashFile -NoTypeInformation

Import-Csv -LiteralPath $HashFile |
    Format-Table -AutoSize
```

This process overwrites the previous manifest and records hashes for the current files. Preserve an earlier manifest separately if you need to retain a historical record.

## 8. Distinguish Registry Changes from Registry Transactions

### Observed behavior

PowerShell successfully created and modified the `LabStatus` value.

### Investigation limitation

These commands demonstrate ordinary Registry modifications:

- `New-Item`
- `New-ItemProperty`
- `Set-ItemProperty`

They do not prove that a transactional Registry operation was used.

### Correct interpretation

Document the operations as controlled Registry changes unless transaction-specific evidence is collected.

A genuine transaction investigation requires an appropriate transaction-aware test harness or other reliable transaction-specific evidence. Sysmon event IDs 12, 13, and 14 alone do not establish transaction commit or rollback.

## 9. Final Validation Checklist

- [ ] Confirm the baseline JSON records the original state.
- [ ] Confirm the action log records creation and modification.
- [ ] Inspect the exported `.reg` file.
- [ ] Regenerate and inspect the JSON state capture.
- [ ] Search the complete Sysmon CSV for the test key.
- [ ] Search Wazuh for the test key and value name.
- [ ] Verify that matching events, if present, have the expected target object.
- [ ] Inspect the cleanup CSV and verify the final key state.
- [ ] Regenerate the SHA-256 manifest after evidence changes.
- [ ] Keep the conclusion limited to what the evidence supports.
