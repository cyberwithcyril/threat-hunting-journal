
# CorpHealth Investigation - Indicators of Compromise (IOCs)

**Case:** Hunt 001 - CorpHealth Operations Activity Review  
**Date Range:** November 23-30, 2025  
**Last Updated:** February 10, 2026

---

## Network Indicators

### External IP Addresses

**Initial Access:**
```
104.164.168.17
- First Seen: 2025-11-23T03:08:31Z
- Last Seen: [Multiple sessions]
- Role: Initial RDP access from Vietnam
- Classification: MALICIOUS - Block at firewall
- Geolocation: Vietnam
- ASN: [To be enriched with threat intel]
```

**Command & Control:**
```
13.228.171.119
- First Seen: 2025-11-23T04:05:00Z (approx)
- Last Seen: 2025-11-30T01:03:17Z+
- Role: AWS-hosted C2 server
- Port: 11746 (TCP)
- Classification: MALICIOUS - Block at firewall
- Infrastructure: AWS (Amazon Web Services)
```

### Internal IP Addresses

**Pivot Point:**
```
10.168.0.7
- Role: Internal Azure VM used as lateral movement pivot
- Status: INVESTIGATE - Potential compromise
- Subnet: Internal Azure VNet
- Next Steps: Full forensic analysis required
```

**Relay/NAT:**
```
100.64.100.6
- Role: CGNAT/relay IP (connection obfuscation)
- Range: 100.64.0.0/10 (Shared Address Space - RFC 6598)
- Significance: Used to mask true attacker origin
```

### Domains & URLs

**Tool Delivery:**
```
https://unresuscitating-donnette-smothery.ngrok-free.dev/revshell.exe
- Service: ngrok dynamic tunnel
- Purpose: Reverse shell executable download
- Classification: MALICIOUS
- Detection: Block ngrok domains OR whitelist-only approach
- Subdomain Pattern: [random-words]-[random-name]-[random-word].ngrok-free.dev
```

**C2 Beaconing:**
```
127.0.0.1:8080
- Type: Localhost proxy/tunnel
- Purpose: Local C2 staging before external connection
- Technique: Proxy to external infrastructure
```

---

## File Indicators

### Malicious Executables

**Reverse Shell:**
```
Filename: revshell.exe
Path 1: C:\Users\ops.maintenance\Downloads\revshell.exe
Path 2: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\revshell.exe
SHA256: [Extract from investigation - Flag 16]
MD5: [To be calculated]
Size: [From file events]
Type: PE32 executable
Behavior: Outbound C2 to 13.228.171.119:11746
Classification: MALICIOUS - Delete on sight
YARA Rule: [To be developed]
```

### Malicious Scripts

**PowerShell Beacon:**
```
Filename: MaintenanceRunner_Distributed.ps1
Path: [To be determined from file events]
SHA256: [To be calculated]
Purpose: C2 beaconing script
Unique To: CH-OPS-WKS02 (not deployed elsewhere)
Classification: MALICIOUS - Remove and hunt for variants
First Execution: 2025-11-23T03:46:08Z
Beacon Destination: 127.0.0.1:8080
```

### Data Staging Files

**Primary Staging:**
```
Filename: inventory_6ECFD4DF.csv
Path: C:\ProgramData\Microsoft\Diagnostics\CorpHealth\inventory_6ECFD4DF.csv
SHA256: 7f6393568e414fc564dad6f49a06a161618b50873404503f82c4447d239f12d8
Purpose: System inventory export (staged for exfiltration)
Classification: SUSPICIOUS - Review contents
Created: 2025-11-23T03:48:15Z (approx)
```

**Secondary Staging:**
```
Filename: inventory_tmp_6ECFD4DF.csv
Path: C:\Users\ops.maintenance\AppData\Local\Temp\CorpHealth\inventory_tmp_6ECFD4DF.csv
SHA256: [DIFFERENT from primary - indicates data transformation]
Purpose: Modified/filtered version of inventory
Classification: SUSPICIOUS - Working copy
Created: 2025-11-23T03:49:30Z (approx)
```

### Credential Files

**Harvested Credentials:**
```
Filename: CH-OPS-WKS02 user-pass.txt
Path: [User desktop or documents - exact path from logs]
Purpose: Plaintext username/password storage
Accessed By: chadmin account at 03:08:35Z
Classification: SENSITIVE - Remove, rotate all credentials
Remediation: Force password reset for all contained accounts
```

---

## Registry Indicators

### Persistence Keys

**Scheduled Task:**
```
Key: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree\CorpHealth_A65E64
Task Name: CorpHealth_A65E64
Created: ~2025-11-23T03:50:00Z
Purpose: Automatic malicious script execution
Classification: MALICIOUS - Delete task
Remediation: Remove task, verify no re-creation
```

**Run Key (Ephemeral):**
```
Key: HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
Value Name: MaintenanceRunner
Value Data: [Path to PowerShell script]
Created: ~2025-11-23T03:52:00Z
Deleted: ~2025-11-23T03:54:00Z
Duration: ~2 minutes (ephemeral persistence)
Classification: MALICIOUS - Monitor for re-creation
```

**Fake Event Source:**
```
Key: HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\EventLog\Application\CorpHealthAgent
Purpose: Create fake event log source for blending in
Created: ~2025-11-23T03:55:00Z
Classification: MALICIOUS - Remove key
```

---

## Account Indicators

### Compromised Accounts

**Initial Access Account:**
```
Username: chadmin
Domain: Local / Workgroup
First Malicious Use: 2025-11-23T03:08:31Z
Source IP: 104.164.168.17
Actions: File access (user-pass.txt), reconnaissance (ipconfig.exe)
Status: COMPROMISED - Force password reset
Remediation: 
  - Immediate password reset
  - Review all logon history
  - Check for persistence under this account
  - Implement MFA
```

**Privileged Operations Account:**
```
Username: ops.maintenance
Domain: Local / Workgroup
Role: Local Administrator on CH-OPS-WKS02
First Malicious Use: ~2025-11-23T03:10:00Z
Actions: Tool deployment, privilege escalation, persistence creation
Design Intent: Automation only (non-interactive)
Actual Use: Interactive RDP session (VIOLATION)
Status: COMPROMISED - Immediate action required
Remediation:
  - Disable account pending investigation
  - Force password reset if re-enabled
  - Restrict to non-interactive logons only
  - Implement privileged access management
  - Alert on ANY interactive use
```

---

## Behavioral Indicators

### Process Patterns

**Encoded PowerShell:**
```
Command Pattern: powershell.exe -EncodedCommand [Base64]
Decoded Example: Write-Output 'token-6D5E4EE08227'
Account: ops.maintenance
Detection: Monitor for -EncodedCommand flag
YARA/Sigma: Create rule for base64-encoded PowerShell
```

**Token Modification:**
```
Event: ProcessPrimaryTokenModified
Process: powershell.exe (PID 4888)
Token SID: S-1-5-21-1605642021-30596605-784192815-1000
Event Payload: "ConfigAdjust"
Detection: Alert on ProcessPrimaryTokenModified events
```

**Localhost Beaconing:**
```
Pattern: Outbound connections to 127.0.0.1:8080
Process: powershell.exe
Script: MaintenanceRunner_Distributed.ps1
Detection: Unusual for maintenance scripts to beacon localhost
Alert: Any PowerShell process connecting to localhost non-standard ports
```

### Network Patterns

**C2 Beacon Characteristics:**
```
Destination: 13.228.171.119:11746
Protocol: TCP
Pattern: Periodic connection attempts over 7+ days
Persistence: Continues despite failures (automated reconnection)
Detection: Sustained connection attempts to same external IP:port
```

**Ngrok Tunnel Usage:**
```
Tool: curl.exe
Destination: *.ngrok-free.dev domains
Purpose: Tool download bypass
Detection: Monitor curl/wget/Invoke-WebRequest to tunnel services
Blocklist: ngrok.io, ngrok-free.dev, localtunnel.me, serveo.net
```

---

## Remote Session Indicators

**Attacker Remote Session Metadata:**
```
Device Name: 对手 (Chinese: "adversary")
Session Type: RemoteInteractive
Protocol: RDP (assumed)
Source IPs: 104.164.168.17, 100.64.100.6, 10.168.0.7
Detection: Alert on device names containing non-ASCII characters
Detection: Alert on interactive logons from service accounts
```

---

## Timeline Anchors

**Critical Timestamps:**
```
2025-11-23T03:08:31.1849379Z - First malicious logon (chadmin from 104.164.168.17)
2025-11-23T03:46:08.400686Z - First C2 beacon attempt (MaintenanceRunner script)
2025-11-23T03:47:21.8529749Z - Privilege escalation event (token modification)
2025-11-30T01:03:17.6985973Z - Latest successful C2 beacon
```

---

## Threat Intelligence Sharing

**Recommended Sharing:**
- Share IOCs with industry ISACs
- Report ngrok abuse to ngrok security team
- Submit malware samples to VirusTotal
- Coordinate with AWS abuse team (C2 infrastructure)
- Share with Vietnam CERT (attacker origin)

**TLP Classification:**
- External IPs, Domains, Hashes: TLP:WHITE (shareable)
- Internal IPs, Account Names: TLP:AMBER (limited sharing)
- Investigation details: TLP:AMBER (need-to-know)

---

**IOC Summary:**
- External IPs: 2
- Internal IPs: 2
- Domains/URLs: 2
- Malicious Files: 5
- Registry Keys: 3
- Compromised Accounts: 2
- Behavioral Patterns: 6+

**Last Updated:** February 10, 2026  
**Analyst:** Jarvis
