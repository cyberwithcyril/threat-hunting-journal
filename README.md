# Threat Hunting & Incident Response Journal

A collection of threat hunting investigations, incident response cases, and detection engineering work focused on real-world attack patterns using KQL, Splunk, and threat intelligence.

## 🎯 About

This repository documents my cybersecurity investigations and threat detection work, including:
- Real-world incident response investigations
- MITRE ATT&CK technique-based threat hunting
- Detection logic development across SIEM platforms
- Cloud security monitoring and analysis
- AI-powered security tools and AI security threat research

**Author:** Cyril Thomas | Cybersecurity Engineer Intern  
**Focus Areas:** SOC Operations, Incident Response, Threat Detection, Cloud Security  
**Tools & Platforms:** Microsoft Sentinel, Defender XDR, KQL, PowerShell, Python

---

## 📚 Investigation Index

### Incident Response Cases
- [001 - CorpHealth Ransomware Attack Investigation](hunts/001-corphealth-ransomware-investigation.md) - *February 10, 2026*  
  Complete end-to-end ransomware investigation from initial phishing access through data exfiltration and encryption. Includes full timeline reconstruction, IOC identification, MITRE ATT&CK mapping, and 8 KQL detection queries.

### Threat Hunts
- Coming Soon: 
- Coming Soon: 
- Coming Soon: 
- Coming Soon: 

---

## 🎯 MITRE ATT&CK Coverage

**Tactics & Techniques Investigated:**

### Initial Access (TA0001)
- ✅ T1566.001 - Phishing: Spearphishing Attachment

### Execution (TA0002)
- ✅ T1059.001 - Command and Scripting Interpreter: PowerShell

### Persistence (TA0003)
- ✅ T1053.005 - Scheduled Task/Job: Scheduled Task

### Credential Access (TA0006)
- ✅ T1003 - OS Credential Dumping
- 📋 T1110.001 - Brute Force: Password Guessing (Planned)

### Lateral Movement (TA0008)
- ✅ T1021.001 - Remote Services: Remote Desktop Protocol
- 📋 T1021.002 - Remote Services: SMB/Windows Admin Shares (Planned)

### Exfiltration (TA0010)
- ✅ T1048 - Exfiltration Over Alternative Protocol

### Impact (TA0040)
- ✅ T1486 - Data Encrypted for Impact

---

**Total Coverage:** 7 techniques across 7 tactics (growing weekly)

**Legend:**
- ✅ Investigated & Documented
- 🔄 Investigation In Progress
- 📋 Planned

---

## 📊 Repository Statistics

- **Total Investigations:** 1
- **Threat Hunts Documented:** 1 (IR case)
- **Detection Rules Created:** 8
- **IOCs Identified:** 15+
- **Platforms Covered:** Microsoft Sentinel (KQL), Defender XDR
- **Attack Chains Documented:** 1 complete kill chain (7 phases)

---

## 🛠️ Technical Skills Demonstrated

### SIEM & Detection Engineering
- Microsoft Sentinel (KQL query development)
- Microsoft Defender for Endpoint
- Log correlation across multiple sources
- Detection rule creation and tuning
- False positive reduction

### Incident Response & Forensics
- Timeline reconstruction and analysis
- Log analysis (Windows Security, Sysmon, Network)
- Malware behavior analysis
- Network traffic analysis (Wireshark)
- Memory forensics concepts
- Chain of custody documentation

### Threat Intelligence
- IOC extraction and documentation
- MITRE ATT&CK framework mapping
- Threat actor TTP analysis
- Intelligence-driven hunting

### Cloud Security
- Azure security monitoring
- Cloud-native SIEM operations
- Identity and access management analysis

### Programming & Automation
- KQL (Kusto Query Language)
- PowerShell scripting
- Python for security automation
- Detection-as-Code principles

---

## 🗂️ Repository Structure
```
threat-hunting-journal/
├── README.md                          # This file
├── hunts/                            # Investigation write-ups
│   └── 001-corphealth-ransomware-investigation.md
├── queries/                          # Detection logic library
│   └── kql/
│       └── ransomware-detection-suite.kql
├── evidence/                         # Supporting investigation materials
│   └── corphealth-case/
│       ├── timeline.md
│       └── iocs.md
├── techniques/                       # MITRE ATT&CK tracking
│   └── mitre-mapping.md
└── resources/                        # Reference materials
    └── tools-and-resources.md
```

---

## 🔍 Featured Investigation

### CorpHealth Ransomware Attack (Case 001)

**Overview:** Multi-stage ransomware attack against healthcare organization resulting in 150+ encrypted systems and 500GB data exfiltration.

**Key Findings:**
- Initial access via spearphishing attachment
- PowerShell-based malware deployment
- Credential dumping (5 accounts compromised including Domain Admin)
- RDP-based lateral movement to 50+ systems
- 9-hour attacker dwell time before ransomware deployment

**Detection Queries Developed:** 8 KQL queries covering all attack phases

**Impact:** Complete investigation demonstrating end-to-end IR capabilities from detection through remediation recommendations.

[→ Read Full Investigation](hunts/001-corphealth-ransomware-investigation.md)

---

## 💡 Methodology & Approach

**Investigation Framework:**
- NIST Incident Response Lifecycle (SP 800-61r2)
- MITRE ATT&CK for threat actor behavior mapping
- Kill Chain analysis for attack reconstruction

**Detection Development:**
- Hypothesis-driven threat hunting
- Low false-positive rate optimization
- Query performance tuning
- Production-ready detection logic

**Documentation Standards:**
- Executive summaries for stakeholder communication
- Technical deep-dives for analyst collaboration
- Reproducible queries and methodologies
- Lessons learned for continuous improvement

---

## 🎓 Learning & Development

This repository reflects my ongoing professional development in cybersecurity, including:

**Academic Work:**
- MS Cybersecurity & Digital Forensics coursework
- Hands-on labs and case studies
- Research into emerging threats

**Professional Experience:**
- Log(N) Pacific: SIEM operations, vulnerability remediation, PowerShell automation
- 100% elimination of critical vulnerabilities
- 90% reduction in high-risk findings

**Continuous Learning:**
- TryHackMe: Active threat hunting practice
- Industry publications: Tracking latest TTPs
- Certification paths: Security+, CySA+, AZ-500

---

## 🔗 Quick Links

### Internal Navigation
- [All Investigations](hunts/)
- [Detection Query Library](queries/)
- [MITRE ATT&CK Mapping](techniques/mitre-mapping.md)
- [Tools & Resources](resources/tools-and-resources.md)

### External Resources
- [Portfolio Website](#) *(Coming Soon)*
- [LinkedIn](https://linkedin.com/in/cyrilkthomas)
- [GitHub](https://github.com/cyberwithcyril)

---

## 📖 How to Use This Repository

**For Fellow Analysts:**
- Queries are provided for educational purposes
- Feel free to adapt detection logic for your environment
- False positive tuning may be needed for your data sources
- Always test in non-production first

**For Students:**
- Each investigation includes methodology and lessons learned
- Use MITRE ATT&CK mapping as learning framework
- Study query construction patterns
- Practice timeline reconstruction techniques

**For Collaboration:**
- Open to feedback on detection logic
- Happy to discuss investigation techniques
- Available for professional networking

---

## 📝 Contributing & Feedback

While this is a personal portfolio repository, I welcome:
- Suggestions for detection rule improvements
- Discussion of investigation methodologies
- Recommendations for additional tools/resources
- Networking with fellow security professionals

**Contact:** [cyrilkthomasest.1991@gmail.com] or connect on LinkedIn

---

## 📜 License

This repository is provided for educational and portfolio purposes.

**Detection Queries:** Use freely with attribution  
**Investigation Methodologies:** Educational use encouraged  
**Original Case Studies:** © 2026 CyrilKThomas - Portfolio work based on academic exercises

---


**Last Updated:** February 10, 2026

**Status:** 🟢 Actively Maintained | Weekly Updates

---

*Building practical cybersecurity skills one investigation at a time.*
