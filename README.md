# 🛡️ Unified 2-Tier SOC & Automated DFIR Platform
### Integrating Centralized SIEM (Wazuh), Remote Live Forensics (Velociraptor), and Offline Triage (KAPE)

[![Platform](https://img.shields.io/badge/Platform-Ubuntu%20%7C%20Windows%2010-blue.svg)](#)
[![SIEM](https://img.shields.io/badge/SIEM-Wazuh%204.x%20%7C%20OpenSearch-0052CC.svg)](#)
[![DFIR](https://img.shields.io/badge/DFIR-Velociraptor%20%7C%20KAPE-orange.svg)](#)
[![Framework](https://img.shields.io/badge/Framework-NIST%20SP%20800--61r2%20%7C%20MITRE%20ATT%26CK-green.svg)](#)

---

## 📌 Executive Summary & Problem Statement

Modern enterprise Security Operations Centers (SOCs) face an overwhelming volume of alerts, where **90%–95%** of detections are false positives or benign administrative anomalies. Escalating raw alerts directly to senior incident responders leads to severe **Alert Fatigue**, delayed Mean Time to Respond (MTTR), and fragmented investigation across multiple disparate tools.

This project designs and implements an end-to-end **Unified 2-Tier SOC & Digital Forensics and Incident Response (DFIR) Framework**:
* **Tier 1 (SOC L1 Analyst):** Continuous real-time detection, contextual validation, automated IoC enrichment, and early de-escalation of benign noise.
* **Tier 2 (DFIR Investigator):** Automated remote evidence acquisition via **Velociraptor VQL**, rapid offline artifact parsing using **KAPE**, maintaining strict forensic chain of custody (RFC 3227 & ISO/IEC 27037).

---

## 🏛️ System Architecture & Network Topology

The solution is architected as an integrated 4-layer model: **Endpoint Telemetry Layer $\rightarrow$ SIEM Processing Layer $\rightarrow$ L1 Triage & SOAR Orchestration $\rightarrow$ L2 DFIR Offline Analysis**:

<p align="center">
  <img src="assets/architecture_4tier.png" alt="4-Tier System Architecture" width="900"/>
  <br>
  <em>Figure 1: Comprehensive 4-Tier Architecture connecting SIEM, SOAR, Velociraptor and KAPE Triage</em>
</p>

```mermaid
graph LR
    subnet["Subnet: 192.168.80.0/24"]

    SIEM["<b>Wazuh SIEM & Velociraptor Server</b><br>IP: 192.168.80.10 (Ubuntu 22.04)<br>• Wazuh Manager / Indexer<br>• Velociraptor Server (Port 8000/8889)<br>• Custom SOC Dashboard"]

    W1["<b>Target Endpoint: WIN10-01</b><br>IP: 192.168.80.31 (Win 10 Pro x64)<br>• Sysmon v15+ (SwiftOnSecurity)<br>• Wazuh Agent (ID: 001)<br>• Velociraptor Agent"]

    W2["<b>Internal Host: WIN10-02</b><br>IP: 192.168.80.32 (Win 10 Pro x64)<br>• Wazuh Agent (ID: 002)<br>• Network Auth Simulation"]

    KALI["<b>Attacker Node: Kali Linux</b><br>IP: 192.168.80.20<br>• Nmap, Metasploit, Hydra<br>• LSASS Memory Dumping"]

    HOST["<b>Offline Forensics Workstation</b><br>• KAPE Triage Analysis<br>• EvtxECmd & Timeline Explorer"]

    SIEM --- subnet
    W1 --- subnet
    W2 --- subnet
    KALI --- subnet
    SIEM -.->|"Secure Artifact Transfer"| HOST
```

---

## ⚙️ Core Technical Capabilities & Implementation

### 1. Centralized Telemetry & Real-Time SIEM Monitoring
* **Telemetry Stack:** Combined `Sysmon v15+` with `Windows Security Event Logs` to detect Process Creation (Event 1), Network Connections (Event 3), Memory Tampering (Event 10), and File Creation (Event 11).
* **Unified Dashboard:** Real-time visibility into alert trends, top attacked endpoints, and MITRE ATT&CK tactic distribution.

<p align="center">
  <img src="assets/wazuh_dashboard.png" alt="Wazuh SIEM Dashboard" width="850"/>
  <br>
  <em>Figure 2: Real-time Security Event Monitoring on Wazuh SIEM Dashboard</em>
</p>

---

### 2. Tier 1 Incident Triage & Inlined Script De-obfuscation
* **False Positive Reduction:** Investigated suspicious PowerShell executions triggered by automated administrative tasks.
* **Inline CyberChef Integration:** Built-in decoding workflows for Base64 / UTF-16LE obfuscated commands directly within the SOC Dashboard to verify execution context without external data leakage.

<p align="center">
  <img src="assets/cyberchef_deobfuscation.png" alt="CyberChef De-obfuscation" width="850"/>
  <br>
  <em>Figure 3: Inline CyberChef Tool resolving Base64 EncodedCommand during L1 Triage</em>
</p>

---

### 3. Multi-Stage Attack Detection: LSASS Dumping via LOLBAS
* **Detection:** SIEM triggers **Level 14 (CRITICAL)** upon detecting unauthorized memory access to `lsass.exe` using legitimate system utilities (LOLBAS).
* **Correlation:** Correlated parent-child execution telemetry across Sysmon Event 10 and Security Event 4688.

<p align="center">
  <img src="assets/lsass_dump_alert.png" alt="LSASS Dump Alert" width="850"/>
  <br>
  <em>Figure 4: Critical Level 14 Alert flagged on SIEM upon memory access to LSASS</em>
</p>

---

### 4. Automated SOAR Evidence Acquisition (Velociraptor)
* **Ticket Escalation:** L1 Analyst bundles correlated alerts into **Incident Ticket #6** and escalates to Tier 2.
* **One-Click Forensic Acquisition:** Tier 2 triggers automated remote collection via Velociraptor API, gathering memory dumps and endpoint triage packages without disrupting production.

<p align="center">
  <img src="assets/velociraptor_soar_triage.png" alt="Velociraptor SOAR Trigger" width="850"/>
  <br>
  <em>Figure 5: Incident Ticket #6 featuring One-Click Velociraptor SOAR Forensic Trigger</em>
</p>

---

### 5. Offline Deep Forensics (KAPE Triage) & Incident Resolution
* **Preserving Chain of Custody:** Extracted `$MFT`, `Prefetch`, `Amcache`, `Registry Hives`, and `Event Logs` hashed via SHA-256 for evidentiary integrity.
* **Forensic Findings:** Bounded execution timelines, confirmed credential compromise scope, and remediated compromised accounts before closing the ticket.

<p align="center">
  <img src="assets/kape_offline_forensics.png" alt="KAPE Offline Forensics" width="850"/>
  <br>
  <em>Figure 6: KAPE Offline Forensics Results revealing Prefetch, Registry, and NTLM artifacts</em>
</p>

<p align="center">
  <img src="assets/ticket_resolved.png" alt="Ticket Resolved" width="850"/>
  <br>
  <em>Figure 7: Incident Ticket #6 successfully updated to RESOLVED following containment verification</em>
</p>

---

## 📂 Repository Structure

```
├── assets/                             # Architecture diagrams & UI screenshots
│   ├── architecture_4tier.png
│   ├── cyberchef_deobfuscation.png
│   ├── kape_offline_forensics.png
│   ├── lsass_dump_alert.png
│   ├── ticket_resolved.png
│   ├── velociraptor_soar_triage.png
│   └── wazuh_dashboard.png
├── docs/
│   └── Unified_SOC_DFIR_Technical_Report.pdf   # Complete 49-Page Technical Specification & Lab Report
├── playbooks/                                  # IR workflows & Triage check-sheets
├── rules/                                      # Custom Wazuh XML rules & Sysmon config
├── velociraptor/                               # Custom VQL artifact definitions
└── README.md                                   # System documentation
```

### 📄 Full Technical Report
For a complete, in-depth walkthrough of the system architecture, mathematical correlation models, and all 8 attack phases with forensic proof, see the full whitepaper:  
👉 [docs/Unified_SOC_DFIR_Technical_Report.pdf](docs/Unified_SOC_DFIR_Technical_Report.pdf) *(Starts directly at Table of Contents)*

---

## 🎓 Technical Credit
* **Author:** Nguyen Thanh Quyet ([@thzhquzt](https://github.com/thzhquzt))
* **Focus:** Defensive Cybersecurity, SOC Operations, Threat Detection & DFIR
