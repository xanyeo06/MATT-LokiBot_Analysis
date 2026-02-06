# Malware_Analysis_LokiBot_Static_Dynamic_Assignment  
**Malware Analysis Tools and Techniques – Academic Assignment**

> ⚠️ **Disclaimer**  
> This project is an **academic malware analysis exercise** conducted in a **controlled laboratory environment**.  
> The malware sample analysed is a **known and publicly documented malware family (LokiBot)** obtained from a malware research repository for **educational purposes only**.  
> This repository does **not distribute malware** and does **not provide operational attack instructions**.

---

## 📘 Project Overview

This repository documents a **comprehensive malware analysis** conducted on a **Windows-based information-stealing Trojan (LokiBot)**.  
The analysis covers **static and dynamic techniques** to understand the malware’s behaviour, capabilities, and impact, as well as **defensive and remediation strategies**.

The goal of this assignment is to demonstrate:
- Practical malware analysis methodology
- Use of industry-standard analysis tools
- Interpretation of malware behaviour
- Defensive response and removal procedures

---

## 🧪 Lab Environment

- **Virtualisation**: VMware Workstation Pro  
- **Analysis OS**: Windows XP (isolated VM)  
- **Network**: Fully isolated lab environment  
- **Sample Type**: Windows executable (Win32)  

No connection to production networks or live systems.

---

## 🦠 Malware Sample Overview

- **Malware Family**: LokiBot  
- **Category**: Information-Stealing Trojan  
- **Targeted Data**:
  - Web browser credentials
  - Email client credentials
  - FTP credentials
  - Windows Credential Manager data
- **Behaviour**:
  - Credential harvesting
  - Registry modification
  - Data encryption
  - Command-and-Control (C2) communication

---

## 🔍 Analysis Methodology

### 1️⃣ Basic Static Analysis
Tools used:
- WinMD5
- PEiD
- BinText
- Dependency Walker
- PEview
- VirusTotal

Focus:
- File integrity verification
- Packer detection
- String extraction
- API import analysis
- Structural inspection
- Malware family identification

---

### 2️⃣ Basic Dynamic Analysis
Tools used:
- Process Monitor (Procmon)
- Process Explorer (ProcExp)
- RegShot
- CaptureBAT
- ApateDNS
- Wireshark

Focus:
- Registry activity
- File system changes
- Loaded DLLs
- Network communications
- Persistence indicators

---

### 3️⃣ Advanced Static Analysis
Tool used:
- IDA Pro

Focus:
- Control flow analysis
- Loop and delay behaviour
- API resolution techniques
- Memory handling routines
- Anti-analysis behaviour

---

### 4️⃣ Advanced Dynamic Analysis
Tool used:
- OllyDbg

Focus:
- Runtime execution flow
- Dynamic API loading
- In-memory unpacking
- Debug-evasion techniques

---

## 📊 Key Findings (High-Level)

- LokiBot is a **credential-stealing Trojan** with broad data-harvesting capabilities  
- The sample is **unpacked**, enabling direct static analysis  
- Uses **dynamic API resolution** to evade detection  
- Modifies registry keys for persistence  
- Encrypts stolen data before exfiltration  
- Communicates directly with a **hardcoded C2 IP** via HTTP  

---

## 🛡️ Defensive Insights

Recommended defences include:
- Endpoint Detection & Response (EDR)
- Blocking known malicious IPs
- Monitoring registry and DLL load activity
- Restricting execution of untrusted binaries
- Network traffic inspection for abnormal outbound connections

---

## 🧹 Malware Removal & Remediation

The report documents a structured remediation approach:
1. System isolation
2. Safe Mode boot
3. Malicious process termination
4. File and registry cleanup
5. Full anti-malware scanning
6. Credential resets
7. Post-incident monitoring

---


## 📜 Ethical Statement

This project strictly follows **ethical cybersecurity practices**:
- Analysis performed on authorised samples only  
- Conducted in isolated lab environments  
- Focused on defence, detection, and understanding malware behaviour  

---

## 📌 Key Learning Outcomes

- Practical malware analysis workflow  
- Understanding of Windows infostealer behaviour  
- Static and dynamic analysis techniques  
- Defensive response and remediation planning  
- Reverse engineering fundamentals
