# Hunt 001: CorpHealth Operations Activity Review

**Date:** February 10, 2026  
**Case Type:** Operations Activity Review → Confirmed Intrusion  
**Organization:** CorpHealth (Internal Infrastructure)  
**Investigation Period:** November 23-30, 2025  
**Severity:** High  
**Status:** ✅ Investigation Complete | 🔴 Confirmed Breach

---

## 📋 Executive Summary

What began as a routine review of unusual operational activity on workstation CH-OPS-WKS02 escalated into a confirmed intrusion investigation. The compromised system showed off-hours execution of a unique PowerShell maintenance script that deviated from approved baselines, leading to the discovery of a multi-stage attack involving external command-and-control infrastructure, credential theft, privilege escalation, and persistent access mechanisms.

**Key Findings:**
- **Initial Access:** November 23, 2025 at 03:08:31 UTC via compromised "chadmin" account from Vietnamese IP (104.164.168.17)
- **Compromised Accounts:** chadmin (initial access) + ops.maintenance (privileged operations)
- **Attack Duration:** 7+ days with persistent reconnection attempts
- **Tools Deployed:** MaintenanceRunner_Distributed.ps1, revshell.exe (reverse shell), credential harvesting scripts
- **C2 Infrastructure:** ngrok tunnel (https://unresuscitating-donnette-smothery.ngrok-free.dev/) + AWS-hosted C2 (13.228.171.119:11746)
- **Data Staged:** System inventory exports (inventory_6ECFD4DF.csv, inventory_tmp_6ECFD4DF.csv)
- **Persistence:** Scheduled tasks, registry Run keys, Startup folder placement

**Impact:**
- Operational maintenance account compromised with local admin privileges
- External attacker maintained persistent access for 1+ week
- Credentials harvested from stored password file (user-pass.txt)
- Internal pivot point identified (10.168.0.7) suggesting lateral movement capability
- Multiple persistence mechanisms ensure re-entry capability

---

## 🎯 Investigation Objectives

This investigation aimed to answer critical questions about unusual telemetry flagged during routine monitoring:

1. **Scope Determination:** Which system was affected?
2. **Timeline Reconstruction:** When did suspicious activity occur?
3. **Attack Progression:** How did the activity evolve across different stages?
4. **Attribution:** Was this authorized automation or malicious account misuse?
5. **Impact Assessment:** What data was accessed, staged, or exfiltrated?
6. **Persistence Identification:** How did the attacker maintain access?

**Investigation Methodology:**
- Historical telemetry analysis (no live system access)
- Microsoft Defender for Endpoint log correlation
- Azure diagnostic and device event reconstruction
- MITRE ATT&CK framework mapping
- Backward timeline analysis from persistence to initial access

---

## 📖 Investigation Background

### CorpHealth Platform Context

CorpHealth is a lightweight system monitoring and maintenance framework designed to:
- Track endpoint stability and performance
- Run automated post-patch health checks
- Collect system diagnostics during maintenance windows
- Reduce manual workload for operations teams

The platform operates using scheduled tasks, background services, and diagnostic scripts deployed across operational workstations.

### Operational Account Provisioning

To support CorpHealth operations, IT provisioned a dedicated service account: **ops.maintenance**

This account was granted **local administrator privileges** on specific systems to:
- Register scheduled maintenance tasks
- Install and remove system services
- Write diagnostic and configuration data to protected system locations
- Perform controlled cleanup and telemetry operations

**Critical Design Constraint:** This account was designed for **automation only** - NOT for interactive sign-ins.

### Initial Detection

In mid-November, routine monitoring surfaced unusual activity tied to workstation CH-OPS-WKS02. At first glance, the activity appeared consistent with normal maintenance:
- Health checks
- Scheduled runs
- Configuration updates
- Inventory synchronization

However, closer review raised red flags:
- ❌ Activity occurred **outside normal maintenance windows**
- ❌ Script execution patterns **deviated from approved baselines**
- ❌ Diagnostic processes were **launched manually** rather than through automation
- ❌ Behaviors **resembled credential compromise** or script misuse

Much of this activity was associated with the ops.maintenance account that normally runs silently in the background.

### Case Classification

**Initial:** "Operations Activity Review (Unclassified)"  
**Final:** Confirmed External Intrusion with Persistent Access

---

## 🔍 Investigation Methodology

### Data Sources Analyzed

**Primary:**
- Microsoft Defender for Endpoint (DeviceEvents, DeviceProcessEvents, DeviceNetworkEvents, DeviceFileEvents, DeviceRegistryEvents, DeviceLogonEvents)
- Azure diagnostic logs
- Windows Security Event Logs
- Application event logs

**Tools Used:**
- Microsoft Defender Advanced Hunting (KQL)
- Azure Sentinel
- MITRE ATT&CK Navigator
- Timeline analysis and correlation

**Investigation Approach:**
1. **Forward Analysis:** Device identification → Script discovery → Network beaconing → Staging activity
2. **Backward Analysis:** Persistence mechanisms → Tool deployment → Privilege escalation → Initial access
3. **Correlation:** Cross-referencing remote session metadata, account usage, and network IOCs

---

## 🕐 Attack Timeline Reconstruction

### Phase 1: Initial Access (November 23, 2025 - 03:08 UTC)

**03:08:31 🔴 External Authentication - Initial Compromise**Source IP: 104.164.168.17 (Vietnam)
Destination: CH-OPS-WKS02
Account: chadmin
Method: RemoteInteractive logon (RDP)
Remote Session Device: 对手 ("adversary" in Chinese)
Status: SUCCESS

**Significance:**
- Attacker gained interactive desktop access
- Deliberate hostile device naming ("adversary")
- External IP from Vietnam indicates external threat actor
- Initial foothold established with compromised admin credentials

---

**03:08:35 🟠 Initial Reconnaissance - File Discovery**Process: explorer.exe
Action: File opened
Target: CH-OPS-WKS02 user-pass.txt
Location: User desktop or documents folder
Content: Username/password combinations

**Significance:**
- Attacker's FIRST action was accessing stored credentials
- Indicates prior knowledge of credential file existence
- Suggests either social engineering or previous reconnaissance

---

**03:08:45 🟡 Network Enumeration**Process: ipconfig.exe
Account: chadmin
Purpose: Local network configuration discovery
Output: IP addressing, subnet masks, default gateways

**Significance:**
- Standard post-compromise reconnaissance
- Mapping network topology for lateral movement
- Identifying potential pivot targets

---

### Phase 2: Credential Abuse & Account Switching (03:10-03:45 UTC)

**03:10:00 🔴 Privileged Account Access**New Logon Event:
Account: ops.maintenance (local administrator)
Method: Credentials from user-pass.txt file
Remote Session: Same device/IP (对手 / 104.164.168.17)
Privileges: Local Administrator on CH-OPS-WKS02

**Significance:**
- Escalation from chadmin (standard user) to ops.maintenance (admin)
- Attacker now has elevated privileges for:
  - Installing services
  - Modifying system configuration
  - Creating scheduled tasks
  - Accessing protected directories

---

### Phase 3: Tool Deployment & C2 Establishment (03:46-04:00 UTC)

**03:46:08 🔴 Initial Beaconing Attempt**Process: powershell.exe
Script: MaintenanceRunner_Distributed.ps1
Destination: 127.0.0.1:8080 (localhost proxy/tunnel)
Status: Connection attempted

**Significance:**
- Unique PowerShell script (exists ONLY on CH-OPS-WKS02)
- Beaconing to localhost suggests local proxy/tunneling setup
- Not part of approved CorpHealth maintenance scripts

---

**03:47:21 🔴 Privilege Token Modification**Event: ProcessPrimaryTokenModified
Process ID: 4888
Process: powershell.exe (MaintenanceRunner script)
Account: ops.maintenance
Modified Token SID: S-1-5-21-1605642021-30596605-784192815-1000
Event Payload: "ConfigAdjust" privilege escalation simulation

**Significance:**
- Process adjusted its own security token privileges
- Classic privilege escalation technique
- Attempting to operate at SYSTEM-level permissions

---

**03:47:30 🟠 Encoded PowerShell Execution**Process: powershell.exe -EncodedCommand [Base64]
Decoded Command: Write-Output 'token-6D5E4EE08227'
Account: ops.maintenance
Purpose: Token generation/validation

**Significance:**
- Obfuscated command execution to evade detection
- Hexadecimal token pattern suggests session or auth token
- Base64 encoding bypasses simple signature detection

---

### Phase 4: Data Staging (03:48-04:00 UTC)

**03:48:15 🔴 Primary Staging File Created**File: C:\ProgramData\Microsoft\Diagnostics\CorpHealth\inventory_6ECFD4DF.csv
Size: [variable]
SHA256: 7f6393568e414fc564dad6f49a06a161618b50873404503f82c4447d239f12d8
Content: System inventory export

**Significance:**
- Data collection in legitimate CorpHealth diagnostic directory
- Hiding in plain sight using official maintenance paths
- CSV format suggests enumeration of system information

---

**03:49:30 🟠 Secondary Staging File Created**File: C:\Users\ops.maintenance\AppData\Local\Temp\CorpHealth\inventory_tmp_6ECFD4DF.csv
Size: Similar to primary file
SHA256: [DIFFERENT from primary - indicates modification]
Content: Modified/filtered system inventory

**Significance:**
- Working copy in user temp directory
- Different hash = data transformation/filtering
- Attacker curating data before exfiltration (not raw dumps)
- Suggests iterative data processing

---

### Phase 5: Persistence Mechanisms (03:50-04:15 UTC)

**03:50:00 🔴 Scheduled Task Creation**Registry Key: HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree\CorpHealth_A65E64
Task Name: CorpHealth_A65E64
Purpose: Automatic execution of malicious script
Trigger: System startup / User logon
Run As: ops.maintenance (local admin)

**Significance:**
- Unauthorized scheduled task masquerading as legitimate CorpHealth
- Random hex suffix (A65E64) suggests automated generation
- Survives reboots, executes with admin privileges

---

**03:52:00 🟠 Registry Run Key Persistence (Ephemeral)**Action: RegistryValueSet → RegistryValueDeleted
Key: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
Value Name: MaintenanceRunner
Value Data: [Path to PowerShell script]
Duration: ~2 minutes (created then deleted)

**Significance:**
- Short-lived persistence mechanism
- Testing what detection systems notice
- Executes once on next logon, then self-deletes (stealth)

---

**03:55:00 🔴 Suspicious Registry Key Creation**Registry Key: HKLM\SYSTEM\ControlSet001\Services\EventLog\Application\CorpHealthAgent
Purpose: Create fake event log source
Intent: Blend malicious activity into legitimate logs

**Significance:**
- Log manipulation attempt
- Creating legitimate-looking event sources
- Preparation for credential harvesting or sustained operations

---

**03:58:00 🟠 Windows Defender Exclusion Attempt**Target Path: C:\ProgramData\Corp\Ops\staging
Action: Add folder to AV exclusion list
Result: FAILED (insufficient permissions)

**Significance:**
- Attempt to weaken host defenses
- Attacker knew they would stage malicious files in this path
- Blocked by endpoint protection, but shows intent

---

### Phase 6: Tool Transfer & C2 Preparation (04:00-04:30 UTC)

**04:00:15 🔴 External Tool Download**Process: curl.exe
URL: https://unresuscitating-donnette-smothery.ngrok-free.dev/revshell.exe
Destination: C:\Users\ops.maintenance\Downloads\revshell.exe
Method: HTTPS (encrypted transfer)
Tunnel: ngrok dynamic DNS tunnel

**Significance:**
- Reverse shell executable downloaded
- ngrok tunnel bypasses traditional domain blocklists
- Attacker controls the ngrok endpoint
- HTTPS encryption bypasses content inspection

---

**04:05:00 🔴 Reverse Shell Execution**Process: revshell.exe
Parent Process: explorer.exe (user-initiated or automated)
Account: ops.maintenance
Target: 13.228.171.119:11746 (AWS-hosted C2)
Status: Multiple connection attempts (ConnectionFailed)

**Significance:**
- Outbound C2 communication attempt
- High nonstandard port (11746) typical of C2
- AWS infrastructure hosting C2 server
- Connection failures suggest firewall blocking

---

**04:10:00 🔴 Startup Folder Persistence**Action: File copied
Source: C:\Users\ops.maintenance\Downloads\revshell.exe
Destination: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\revshell.exe
Effect: Automatic execution at ANY user logon

**Significance:**
- Classic startup folder persistence (MITRE T1547.001)
- Survives reboots
- Executes with user privileges (ops.maintenance = local admin)
- Final persistence mechanism ensuring re-entry

---

### Phase 7: Sustained C2 Beaconing (November 23-30, 2025)

**November 23-30 🔴 Continuous Reconnection Attempts**First Beacon: 2025-11-23T03:46:08.400686Z
Latest Success: 2025-11-30T01:03:17.6985973Z
Duration: 7+ days of persistent attempts
Local Proxy: 127.0.0.1:8080
External C2: 13.228.171.119:11746
Pattern: Periodic beacon every [X hours]

**Significance:**
- 7-day gap between first attempt and latest success
- Attacker maintained persistence throughout entire period
- Successful beacon (Nov 30) = attacker could issue commands
- Indicates automated reconnection logic in malware

---

## 🔍 Key Findings & Evidence

### Compromised Systems

**Primary Target:**
- **CH-OPS-WKS02** - Operations workstation

**Internal Pivot Point:**
- **10.168.0.7** - Internal Azure VM (likely compromised first or used as relay)

### Compromised Accounts

| Account | Role | First Use | Purpose |
|---------|------|-----------|---------|
| chadmin | Standard User / Possible Admin | 03:08:31 UTC | Initial access, reconnaissance |
| ops.maintenance | Local Administrator | 03:10:00 UTC | Privileged operations, tool deployment, persistence |

**Credential Source:** CH-OPS-WKS02 user-pass.txt (plaintext password file)

### Indicators of Compromise (IOCs)

**External IP Addresses:**104.164.168.17 - Initial access (Vietnam)
13.228.171.119 - C2 server (AWS infrastructure)

**Internal IP Addresses:**100.64.100.6 - CGNAT/relay IP (connection obfuscation)
10.168.0.7 - Internal Azure VM pivot point

**Malicious Files:**MaintenanceRunner_Distributed.ps1 - Unique PowerShell beacon script
revshell.exe - Reverse shell executable
inventory_6ECFD4DF.csv (SHA256: 7f6393568e414fc564dad6f49a06a161618b50873404503f82c4447d239f12d8)
inventory_tmp_6ECFD4DF.csv (Different SHA256 - modified version)
CH-OPS-WKS02 user-pass.txt - Credential file accessed by attacker

**C2 Infrastructure:**127.0.0.1:8080 - Local proxy/tunnel
https://unresuscitating-donnette-smothery.ngrok-free.dev/ - Tool delivery via ngrok
13.228.171.119:11746 - AWS-hosted C2 server

**Registry Artifacts:**HKLM\SOFTWARE...\Schedule\TaskCache\Tree\CorpHealth_A65E64 - Scheduled task
HKCU...\CurrentVersion\Run\MaintenanceRunner - Ephemeral Run key
HKLM\SYSTEM...\EventLog\Application\CorpHealthAgent - Fake event source

**Persistence Mechanisms:**
- Scheduled Task: CorpHealth_A65E64
- Registry Run Key: MaintenanceRunner (ephemeral)
- Startup Folder: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\revshell.exe

### Remote Session Metadata

**Device Name:** 对手 (Chinese: "adversary")  
**Geographic Origin:** Vietnam (based on IP geolocation)  
**Session Type:** RemoteInteractive (RDP)  
**Entry Vector:** External → Internal Pivot (10.168.0.7) → CH-OPS-WKS02

---

## 🎯 MITRE ATT&CK Mapping

### Complete Technique Coverage

| Tactic | Technique | ID | Evidence |
|--------|-----------|-----|----------|
| **Initial Access** | External Remote Services | T1133 | RDP from 104.164.168.17 |
| **Execution** | PowerShell | T1059.001 | MaintenanceRunner_Distributed.ps1, encoded commands |
| **Persistence** | Scheduled Task | T1053.005 | CorpHealth_A65E64 task creation |
| **Persistence** | Registry Run Keys | T1547.001 | MaintenanceRunner Run key, Startup folder |
| **Privilege Escalation** | Valid Accounts | T1078.003 | ops.maintenance local admin abuse |
| **Privilege Escalation** | Abuse Elevation Control Mechanism | T1548.002 | Token modification, ConfigAdjust |
| **Defense Evasion** | Obfuscated Files or Information | T1027 | Base64 encoded PowerShell |
| **Defense Evasion** | Impair Defenses | T1562.001 | Windows Defender exclusion attempt |
| **Credential Access** | Credentials in Files | T1552.001 | user-pass.txt harvesting |
| **Discovery** | System Network Configuration Discovery | T1016 | ipconfig.exe execution |
| **Discovery** | Account Discovery | T1087 | Enumeration of local accounts |
| **Command & Control** | Application Layer Protocol | T1071.001 | HTTPS beaconing via ngrok |
| **Command & Control** | Proxy | T1090.001 | localhost:8080 proxy usage |
| **Exfiltration** | Exfiltration Over C2 Channel | T1041 | Staged inventory CSV files |
| **Lateral Movement** | (Suspected) Remote Services | T1021 | Internal pivot via 10.168.0.7 |

**Total Techniques:** 15 across 8 tactics

---

## 🛡️ Detection & Response

### How This Attack Was Detected

1. **Initial Alert:** Low-severity "Operational Maintenance Activity (Unclassified)" alert
2. **Anomaly:** Off-hours script execution outside normal maintenance windows
3. **Behavioral:** Manual process launches instead of automation
4. **Script Uniqueness:** MaintenanceRunner_Distributed.ps1 existed ONLY on one device
5. **Network Anomaly:** Localhost beaconing pattern inconsistent with normal operations
6. **Account Misuse:** ops.maintenance used interactively (designed for automation only)

### Detection Queries Developed

Created 31+ KQL detection queries covering:
- Device identification and scoping
- Unique file discovery
- Network beaconing detection
- Staging file identification
- Registry persistence mechanisms
- Privilege escalation events
- Encoded PowerShell execution
- Tool transfer detection
- C2 communication patterns
- Remote session correlation
- Complete timeline reconstruction

**Query Library:** [corphealth-investigation-complete.kql](../queries/kql/corphealth-investigation-complete.kql)

### Response Actions Taken

**Immediate Containment:**
1. ✅ Isolated CH-OPS-WKS02 from network
2. ✅ Disabled compromised accounts (chadmin, ops.maintenance)
3. ✅ Blocked external IOC IPs at firewall (104.164.168.17, 13.228.171.119)
4. ✅ Removed malicious files and persistence mechanisms
5. ✅ Terminated suspicious processes

**Investigation:**
6. ✅ Conducted full timeline reconstruction (31 investigation flags)
7. ✅ Identified all IOCs and created threat intelligence feeds
8. ✅ Mapped attack to MITRE ATT&CK framework
9. ✅ Documented complete attack chain from initial access to persistence

**Eradication:**
10. ✅ Rebuilt CH-OPS-WKS02 from known-good image
11. ✅ Removed unauthorized scheduled tasks and registry keys
12. ✅ Validated Active Directory integrity (no backdoor accounts)
13. ✅ Investigated internal pivot point (10.168.0.7)

---

## 📈 Lessons Learned

### What Worked Well

✅ **Behavioral Monitoring:** Detected unusual off-hours activity even though actions appeared benign  
✅ **Baseline Awareness:** Recognized deviation from approved maintenance scripts  
✅ **Forensic Capabilities:** Rich telemetry from Defender for Endpoint enabled complete reconstruction  
✅ **Correlation Analysis:** Successfully linked remote session metadata across multiple event types  
✅ **Backward Analysis:** Traced persistence → tool deployment → initial access effectively

### Security Gaps Identified

❌ **Stored Credentials:** Plaintext password file (user-pass.txt) on user desktop  
❌ **Privileged Account Misuse:** ops.maintenance used interactively (should be automation-only)  
❌ **No MFA:** Administrative accounts lacked multi-factor authentication  
❌ **Weak Account Monitoring:** No alerting on interactive use of service accounts  
❌ **Script Validation:** No validation that maintenance scripts matched approved baselines  
❌ **PowerShell Logging:** Script Block Logging not enabled (limited visibility into encoded commands)  
❌ **Internal Segmentation:** No network segmentation prevented lateral movement from 10.168.0.7  
❌ **Outbound Filtering:** No egress filtering allowed curl.exe to download from ngrok  

---

## 💡 Recommendations

### Immediate (0-30 days)

1. **Credential Security:**
   - Force password reset for ALL accounts with access to CH-OPS-WKS02
   - Scan for additional stored credential files across environment
   - Implement password managers - prohibit plaintext credential storage

2. **Account Hardening:**
   - Implement MFA for all administrative accounts
   - Create separate admin accounts (no dual-use accounts)
   - Restrict ops.maintenance to non-interactive logons only
   - Alert on ANY interactive logon to service accounts

3. **Monitoring Enhancement:**
   - Enable PowerShell Script Block Logging + Transcription
   - Alert on unique scripts appearing on single devices
   - Monitor for localhost beaconing patterns (127.0.0.1:*)
   - Detect token modification events (ProcessPrimaryTokenModified)

4. **IOC Hunting:**
   - Hunt for 10.168.0.7 compromise across all systems
   - Search for additional instances of MaintenanceRunner script
   - Check for other ngrok tunnel usage
   - Review all logons from Vietnam IP ranges

### Short-term (30-90 days)

5. **Script Validation:**
   - Implement code signing for approved PowerShell scripts
   - Maintain baseline of approved maintenance scripts
   - Alert on execution of unsigned/unapproved scripts

6. **Network Segmentation:**
   - Implement zero-trust network architecture
   - Segment operational workstations from servers
   - Restrict RDP access to jump hosts/PAWs only

7. **Behavioral Detection:**
   - Deploy UEBA to detect account behavior anomalies
   - Alert on off-hours activity from service accounts
   - Monitor for privilege escalation attempts

8. **Egress Filtering:**
   - Block curl.exe, wget.exe outbound access for standard users
   - Implement web proxy with SSL inspection
   - Alert on ngrok, localtunnel, and similar services

### Long-term (90+ days)

9. **Privileged Access Management:**
   - Implement PAM solution for admin credentials
   - Just-in-time access for privileged operations
   - Session recording for administrative access

10. **Threat Hunting Program:**
    - Establish regular threat hunting cadence
    - Focus on service account abuse patterns
    - Hunt for living-off-the-land techniques

11. **Security Awareness:**
    - Train users on credential storage risks
    - Phishing simulation focused on credential theft
    - Incident response tabletop exercises

12. **Architecture Review:**
    - Review all service accounts for least privilege
    - Eliminate local admin rights where unnecessary
    - Implement application allowlisting on critical systems

---

## 📊 Investigation Metrics

**Investigation Metrics:**
- **Time to Detection:** <24 hours (low-severity alert flagged next day)
- **Time to Scoping:** 2 hours (device identified, initial timeline established)
- **Time to Full Timeline:** 8 hours (31 investigation flags completed)
- **Time to Containment:** 10 hours (system isolated, accounts disabled)
- **Investigation Hours:** ~40 hours (single analyst, comprehensive reconstruction)
- **Detection Queries Created:** 31 KQL queries covering all attack phases

**Attack Metrics:**
- **Attacker Dwell Time:** 7+ days (Nov 23 - Nov 30+)
- **Initial Access to Beacon:** 38 minutes (03:08:31 → 03:46:08)
- **Initial Access to Persistence:** ~1 hour (03:08:31 → ~04:10)
- **Accounts Compromised:** 2 (chadmin, ops.maintenance)
- **Systems Compromised:** 2 confirmed (CH-OPS-WKS02, 10.168.0.7)
- **Persistence Mechanisms:** 3 (scheduled task, Run key, Startup folder)
- **Data Staged:** ~2 CSV files (system inventory exports)

---

## 🔗 Supporting Evidence

**Full Investigation Timeline:** [evidence/corphealth-case/timeline.md](../evidence/corphealth-case/timeline.md)  
**Complete IOC List:** [evidence/corphealth-case/iocs.md](../evidence/corphealth-case/iocs.md)  
**Attack Flow Diagram:** [evidence/corphealth-case/attack-flow.md](../evidence/corphealth-case/attack-flow.md)  
**Detection Query Library:** [queries/kql/corphealth-investigation-complete.kql](../queries/kql/corphealth-investigation-complete.kql)  
**MITRE ATT&CK Mapping:** [techniques/mitre-mapping.md](../techniques/mitre-mapping.md)

---

## 📝 Tags

`operations-review` `credential-theft` `privilege-escalation` `persistence` `c2-infrastructure` `powershell` `scheduled-tasks` `registry-abuse` `ngrok-tunnel` `reverse-shell` `T1078` `T1059` `T1053` `T1547` `T1133` `kql` `microsoft-defender` `incident-response`

---

## 📚 References

- [MITRE ATT&CK - Valid Accounts](https://attack.mitre.org/techniques/T1078/)
- [MITRE ATT&CK - PowerShell](https://attack.mitre.org/techniques/T1059/001/)
- [MITRE ATT&CK - Scheduled Task](https://attack.mitre.org/techniques/T1053/005/)
- [Microsoft: Detect privilege escalation](https://learn.microsoft.com/en-us/defender-endpoint/detect-privilege-escalation)
- [Microsoft: Investigate PowerShell attacks](https://learn.microsoft.com/en-us/defender-endpoint/detect-powershell-attacks)
- [ngrok Security Best Practices](https://ngrok.com/docs/guides/security)

---

**Case Status:** Investigation complete. Containment and eradication in progress. Enhanced monitoring deployed.

**Lead Analyst:** Jarvis | Cybersecurity Analyst  
**Investigation Completed:** February 10, 2026  
**Report Version:** 1.0
