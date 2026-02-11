# CorpHealth Attack Flow Diagram

**Case:** Hunt 001  
**Device:** CH-OPS-WKS02  
**Attacker Origin:** Vietnam (104.164.168.17)

---

## ASCII Attack Flow
```
┌─────────────────────────────────────────────────────────────────┐
│                      ATTACK FLOW DIAGRAM                        │
│                  CorpHealth Operations Review                   │
└─────────────────────────────────────────────────────────────────┘

PHASE 1: INITIAL ACCESS
═══════════════════════

    External Attacker
    (Vietnam)
    104.164.168.17
          │
          │ [1] RDP Connection
          │     Remote Session Device: 对手
          │     03:08:31 UTC
          ▼
   ┌──────────────────┐
   │  CH-OPS-WKS02    │
   │  (Operations)    │◄──── Internal Pivot?
   │                  │      10.168.0.7
   └──────────────────┘
          │
          │ [2] Successful Logon
          │     Account: chadmin
          ▼
   ┌──────────────────┐
   │ Interactive      │
   │ Desktop Session  │
   └──────────────────┘


PHASE 2: RECONNAISSANCE
════════════════════════

    chadmin Session
          │
          │ [3] File Access
          │     03:08:35 UTC
          ▼
   ┌──────────────────┐
   │  user-pass.txt   │
   │  (Credentials)   │
   └──────────────────┘
          │
          │ [4] Extracted Credentials:
          │     - chadmin password
          │     - ops.maintenance password
          │     - [other accounts]
          ▼
   ┌──────────────────┐
   │  ipconfig.exe    │
   │  (Network Enum)  │
   └──────────────────┘


PHASE 3: PRIVILEGE ESCALATION
═══════════════════════════════

    Credential Abuse
          │
          │ [5] Account Switch
          │     03:10:00 UTC
          ▼
   ┌──────────────────┐
   │ ops.maintenance  │
   │ (Local Admin)    │
   └──────────────────┘


PHASE 4: EXECUTION & C2
════════════════════════

    ops.maintenance
          │
          │ [6] Script Deployment
          │     03:46:08 UTC
          ▼
   ┌──────────────────────────┐
   │ MaintenanceRunner_       │
   │ Distributed.ps1          │
   │ (Unique to this device)  │
   └──────────────────────────┘
          │
          │ [7] Beacon Localhost
          ▼
   ┌──────────────────┐
   │ 127.0.0.1:8080   │
   │ (Local Proxy)    │
   └──────────────────┘
          │
          │ [8] External Beacon
          │     (Failed initially)
          ▼
   ┌──────────────────┐
   │ External C2      │
   │ (Connection      │
   │  pending)        │
   └──────────────────┘


PHASE 5: PRIVILEGE MANIPULATION
═════════════════════════════════

    powershell.exe
    (PID 4888)
          │
          │ [9] Token Modification
          │     03:47:21 UTC
          ▼
   ┌──────────────────────────┐
   │ ProcessPrimaryToken      │
   │ Modified                 │
   │ SID: S-1-5-21-...1000   │
   │ Event: ConfigAdjust      │
   └──────────────────────────┘
          │
          │ [10] Encoded Execution
          ▼
   ┌──────────────────────────┐
   │ powershell.exe           │
   │ -EncodedCommand [Base64] │
   │ Decoded: Write-Output    │
   │ 'token-6D5E4EE08227'    │
   └──────────────────────────┘


PHASE 6: DATA STAGING
═══════════════════════

    Elevated PowerShell
          │
          │ [11] File Creation
          │      03:48:15 UTC
          ▼
   ┌────────────────────────────────┐
   │ C:\ProgramData\...\CorpHealth\ │
   │ inventory_6ECFD4DF.csv         │
   │ SHA256: 7f639356...           │
   └────────────────────────────────┘
          │
          │ [12] Modified Copy
          │      03:49:30 UTC
          ▼
   ┌────────────────────────────────┐
   │ C:\Users\ops.maintenance\...   │
   │ inventory_tmp_6ECFD4DF.csv     │
   │ SHA256: [DIFFERENT]            │
   └────────────────────────────────┘


PHASE 7: PERSISTENCE
═════════════════════

    Three Mechanisms:

    [13] Scheduled Task           [14] Registry Run Key      [15] Fake Event Source
         03:50:00 UTC                  03:52:00 UTC               03:55:00 UTC
              │                             │                          │
              ▼                             ▼                          ▼
   ┌─────────────────────┐      ┌─────────────────────┐    ┌────────────────────┐
   │ HKLM\...\Schedule\  │      │ HKCU\...\Run\       │    │ HKLM\SYSTEM\...\   │
   │ TaskCache\Tree\     │      │ MaintenanceRunner   │    │ EventLog\App\      │
   │ CorpHealth_A65E64   │      │ (Ephemeral - 2 min) │    │ CorpHealthAgent    │
   └─────────────────────┘      └─────────────────────┘    └────────────────────┘


PHASE 8: DEFENSE EVASION
══════════════════════════

    [16] AV Exclusion Attempt
         03:58:00 UTC
              │
              │ Windows Defender
              │ Exclusion Path:
              ▼
   ┌─────────────────────────────┐
   │ C:\ProgramData\Corp\Ops\    │
   │ staging                     │
   │ Status: FAILED              │
   │ (Insufficient Permissions)  │
   └─────────────────────────────┘


PHASE 9: TOOL TRANSFER
═══════════════════════

    curl.exe
         │
         │ [17] HTTPS Download
         │      04:00:15 UTC
         ▼
   ┌──────────────────────────────────────────┐
   │ https://unresuscitating-donnette-        │
   │ smothery.ngrok-free.dev/revshell.exe     │
   │ (ngrok Dynamic Tunnel)                   │
   └──────────────────────────────────────────┘
         │
         │ Downloaded to:
         ▼
   ┌──────────────────────────────────────────┐
   │ C:\Users\ops.maintenance\Downloads\      │
   │ revshell.exe                             │
   └──────────────────────────────────────────┘


PHASE 10: C2 ESTABLISHMENT
════════════════════════════

    [18] Execution
         04:05:00 UTC
              │
         explorer.exe
              │
              ▼
   ┌─────────────────────┐
   │  revshell.exe       │
   └─────────────────────┘
              │
              │ [19] Outbound Connection
              │      Multiple Attempts
              ▼
   ┌─────────────────────┐
   │ 13.228.171.119      │
   │ Port: 11746         │
   │ (AWS-hosted C2)     │
   │ Status: Failed      │
   │ (Firewall blocked)  │
   └─────────────────────┘


PHASE 11: FINAL PERSISTENCE
═════════════════════════════

    [20] File Copy
         04:10:00 UTC
              │
              ▼
   ┌──────────────────────────────────────────┐
   │ C:\ProgramData\Microsoft\Windows\        │
   │ Start Menu\Programs\StartUp\             │
   │ revshell.exe                             │
   │                                          │
   │ Effect: Auto-execute at ANY user logon  │
   └──────────────────────────────────────────┘


PHASE 12: SUSTAINED ACCESS
════════════════════════════

    Nov 23 - Nov 30 (7+ days)
         │
         │ Periodic Reconnection
         │ First: 03:46:08Z
         │ Latest Success: Nov 30 01:03:17Z
         ▼
   ┌─────────────────────┐
   │ 127.0.0.1:8080      │
   │ (Local Proxy)       │
   └─────────────────────┘
         │
         │ External Beacon
         ▼
   ┌─────────────────────┐
   │ C2 Infrastructure   │
   │ (Maintained Access) │
   └─────────────────────┘
```

---

## Kill Chain Mapping
```
┌────────────────────────────────────────────────────────────┐
│            LOCKHEED MARTIN CYBER KILL CHAIN                │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  1. RECONNAISSANCE                                         │
│     └─> (Pre-attack, not observed in logs)                │
│                                                            │
│  2. WEAPONIZATION                                          │
│     └─> MaintenanceRunner script, revshell.exe            │
│                                                            │
│  3. DELIVERY                                               │
│     └─> RDP from external IP (104.164.168.17)            │
│     └─> ngrok tunnel (tool delivery)                      │
│                                                            │
│  4. EXPLOITATION                                           │
│     └─> Compromised chadmin credentials                   │
│     └─> Stored password file (user-pass.txt)             │
│                                                            │
│  5. INSTALLATION                                           │
│     └─> MaintenanceRunner.ps1                            │
│     └─> revshell.exe → Startup folder                    │
│     └─> Scheduled task: CorpHealth_A65E64                │
│                                                            │
│  6. COMMAND & CONTROL                                      │
│     └─> Localhost:8080 → External C2                     │
│     └─> 13.228.171.119:11746 (AWS)                       │
│     └─> 7+ days of sustained beaconing                   │
│                                                            │
│  7. ACTIONS ON OBJECTIVES                                  │
│     └─> Credential harvesting                             │
│     └─> System inventory collection                       │
│     └─> Persistent access establishment                   │
│     └─> (Potential staging for lateral movement)         │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## MITRE ATT&CK Heat Map
```
TACTICS OBSERVED:

Initial Access        ████████░░ 80%  (External Remote Services)
Execution             ██████████ 100% (PowerShell, Binaries)
Persistence           ██████████ 100% (3 mechanisms)
Privilege Escalation  ████████░░ 80%  (Valid Accounts, Token Mod)
Defense Evasion       ██████░░░░ 60%  (Obfuscation, AV bypass attempt)
Credential Access     ████████░░ 80%  (Credentials in Files)
Discovery             ██████░░░░ 60%  (Network, Account enum)
Lateral Movement      ████░░░░░░ 40%  (Suspected, via 10.168.0.7)
Command & Control     ██████████ 100% (C2 infrastructure established)
Exfiltration          ████░░░░░░ 40%  (Staged, not confirmed exfil)
Impact                ░░░░░░░░░░ 0%   (No destructive actions observed)

TECHNIQUES: 15 across 8 tactics
```

---

## Network Flow Diagram
```
ATTACKER INFRASTRUCTURE                VICTIM NETWORK
══════════════════════                 ══════════════

104.164.168.17 ─────────────────────> CH-OPS-WKS02
(Vietnam)              │ RDP           (Operations)
                       │ 03:08:31           │
                       │                    │
                       │                    │ Internal Pivot?
                       │                    ├──────> 10.168.0.7
                       │                    │        (Azure VM)
                       │                    │
                       ▼                    │
            ┌──────────────────┐           │
            │  chadmin         │           │
            │  → ops.maintenance│          │
            └──────────────────┘           │
                       │                    │
                       │ Deploy Script      │
                       ▼                    │
            ┌──────────────────┐           │
            │ MaintenanceRunner│           │
            │ .ps1             │           │
            └──────────────────┘           │
                       │                    │
                       ├─> 127.0.0.1:8080  │
                       │   (Local Proxy)    │
                       │                    │
ngrok Tunnel           │                    │
============           │                    │
unresuscitating- <─────┤                   │
donnette-smothery.     │ curl download      │
ngrok-free.dev         │ 04:00:15           │
     │                 ▼                    │
     │          ┌──────────────────┐       │
     └────────> │  revshell.exe    │       │
                └──────────────────┘       │
                       │                    │
                       │ Outbound C2        │
                       │ 04:05:00           │
                       ▼                    │
13.228.171.119 <───────────────────────────┘
Port: 11746         Failed (Firewall)
(AWS-hosted C2)     Success: Nov 30 01:03:17
```

---

## Detection Points
```
WHERE DETECTION COULD HAVE STOPPED THE ATTACK:
═══════════════════════════════════════════════

[1] INITIAL ACCESS
    ❌ No MFA on RDP
    ❌ No geofencing (Vietnam → corporate assets)
    ✅ Could alert: Non-ASCII device names ("对手")

[2] CREDENTIAL ACCESS
    ❌ Plaintext password file on desktop
    ❌ No file access monitoring
    ✅ Could alert: Access to *pass*.txt files

[3] ACCOUNT ABUSE
    ❌ Service account used interactively
    ❌ No alert on ops.maintenance interactive logon
    ✅ Could alert: Interactive use of automation accounts

[4] SCRIPT EXECUTION
    ❌ Unique script not validated against baseline
    ❌ PowerShell Script Block Logging disabled
    ✅ Could alert: Unique scripts on single devices

[5] C2 BEACONING
    ❌ Localhost beaconing allowed
    ❌ No egress filtering
    ✅ Could alert: PowerShell → localhost:8080

[6] PRIVILEGE ESCALATION
    ❌ Token modification not monitored
    ✅ Could alert: ProcessPrimaryTokenModified events

[7] TOOL TRANSFER
    ❌ curl.exe allowed to ngrok domains
    ❌ No executable download blocking
    ✅ Could alert: curl/wget to tunnel services

[8] PERSISTENCE
    ❌ Scheduled task creation not monitored
    ❌ Startup folder writes not alerted
    ✅ Could alert: Unauthorized scheduled tasks
    ✅ Could alert: .exe files → Startup folder

DETECTION OPPORTUNITIES: 8+ points where attack could have been stopped
```

---

**Diagram Created By:** Cyril Thomas
**Date:** February 10, 2026  
**Purpose:** Visual attack reconstruction for stakeholder briefing
