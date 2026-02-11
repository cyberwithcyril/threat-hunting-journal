
## CorpHealth Investigation (Hunt 001)

### Initial Access (TA0001)
- ✅ T1133 - External Remote Services (RDP from Vietnam)

### Execution (TA0002)
- ✅ T1059.001 - PowerShell (MaintenanceRunner script, encoded commands)

### Persistence (TA0003)
- ✅ T1053.005 - Scheduled Task/Job
- ✅ T1547.001 - Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder

### Privilege Escalation (TA0004)
- ✅ T1078.003 - Valid Accounts: Local Accounts
- ✅ T1548.002 - Abuse Elevation Control Mechanism: Bypass UAC (token modification)

### Defense Evasion (TA0005)
- ✅ T1027 - Obfuscated Files or Information (Base64 encoding)
- ✅ T1562.001 - Impair Defenses: Disable or Modify Tools (AV exclusion attempt)

### Credential Access (TA0006)
- ✅ T1552.001 - Unsecured Credentials: Credentials In Files (user-pass.txt)

### Discovery (TA0007)
- ✅ T1016 - System Network Configuration Discovery (ipconfig)
- ✅ T1087 - Account Discovery

### Lateral Movement (TA0008)
- 📋 T1021 - Remote Services (Suspected via 10.168.0.7)

### Command & Control (TA0011)
- ✅ T1071.001 - Application Layer Protocol: Web Protocols (HTTPS to ngrok)
- ✅ T1090.001 - Proxy: Internal Proxy (127.0.0.1:8080)

### Exfiltration (TA0010)
- ✅ T1041 - Exfiltration Over C2 Channel (Staged inventory files)

**Total: 15 techniques across 8 tactics**
