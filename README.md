# 🔐 SOC Analyst Home Lab

[![Lab Validation](https://github.com/sandeepmothukuri/soc-lab/actions/workflows/validate.yml/badge.svg)](https://github.com/sandeepmothukuri/soc-lab/actions)
[![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-0066CC?logo=wazuh&logoColor=white)](https://wazuh.com/)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-red)](https://attack.mitre.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Stars](https://img.shields.io/github/stars/sandeepmothukuri/soc-lab?style=social)](https://github.com/sandeepmothukuri/soc-lab)

> **Hands-on SOC analyst lab** — brute-force attack simulation, endpoint detection with Wazuh SIEM + Sysmon, and MITRE ATT&CK mapping. Built from scratch to demonstrate real L1/L2 SOC detection workflows.

---

## ⚡ Quick Demo

| | |
|---|---|
| **Attack** | Hydra brute-force on RDP |
| **Detection** | Event ID 4625 correlation |
| **SIEM** | Wazuh |
| **Result** | Attack detected, no successful compromise |
| **MITRE** | T1110 — Brute Force |

---

## 📌 Overview

Detected and analyzed a brute-force attack using **Wazuh SIEM**, **Sysmon** telemetry, and **MITRE ATT&CK** mapping in a self-built SOC home lab.

---

## 🏗️ Architecture

```
Kali Linux (Attacker)
        │
        ▼  Hydra / Nmap
Windows 10 Endpoint
   ├─ Sysmon (process/network telemetry)
   └─ Wazuh Agent
        │
        ▼
Wazuh Manager (Ubuntu)
   ├─ Elasticsearch (log storage)
   └─ Kibana Dashboard (visualization)
```

---

## ⚙️ Installation Guide

### Prerequisites

- VirtualBox → https://www.virtualbox.org/
- VMware → https://www.vmware.com/
- Minimum: 8 GB RAM, 100 GB storage

### Deploy Wazuh SIEM

```bash
sudo apt update && sudo apt upgrade -y
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

Access dashboard: `https://<WAZUH-IP>`

### Windows Endpoint — Install Sysmon

```powershell
sysmon.exe -i sysmon-config.xml
```

### Windows Endpoint — Install Wazuh Agent

Download: https://packages.wazuh.com/4.x/windows/wazuh-agent.msi

Edit `C:\Program Files (x86)\ossec-agent\ossec.conf`:
```xml
<address>WAZUH-IP</address>
```

```powershell
net start wazuh
```

### Connect Agent to Manager

```bash
sudo /var/ossec/bin/manage_agents
```

---

## ⚔️ Attack Simulation

```bash
# Network scan
nmap -sS <target-ip>

# Brute force RDP
hydra -l admin -P rockyou.txt rdp://<target-ip>
```

---

## 🚨 Detection Workflow

### Key Windows Event IDs

| Event ID | Description |
|---|---|
| 4625 | Failed login attempt |
| 4624 | Successful login |
| 4740 | Account lockout |
| 4672 | Privilege assignment |
| 4663 | File access |
| Sysmon 1 | Process creation |
| Sysmon 3 | Network connection |

### Detection Logic

- **Trigger:** 5+ failed logins from the same source IP within 60 seconds
- **Rule ID:** 100001 (custom)
- **MITRE:** T1110 — Brute Force

### Custom Wazuh Rule

```xml
<rule id="100001" level="10">
  <if_sid>18107</if_sid>
  <description>Brute force attack detected</description>
  <mitre>T1110</mitre>
</rule>
```

```bash
sudo systemctl restart wazuh-manager
```

---

## 📅 Incident Timeline

| Step | Event |
|---|---|
| 1 | Nmap scan detected (T1046) |
| 2 | Hydra brute-force initiated |
| 3 | Multiple Event ID 4625 alerts fired |
| 4 | Wazuh custom rule triggered |
| 5 | Logs correlated in SIEM |
| 6 | No 4624 found — no successful compromise |
| 7 | Incident closed as contained |

---

## 🎯 MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Brute Force | T1110 | Multiple failed login attempts |
| Network Scanning | T1046 | Nmap port scan |
| Command Execution | T1059 | Shell commands on attacker |
| Valid Accounts | T1078 | Target for credential access |

---

## 📸 Lab Evidence

### 🧠 SOC Architecture

![Architecture](images/architecture/soc-architecture/architecture.png)
![Elastic Stack](images/architecture/soc-architecture/elastic-stack-integration.png)
![ELK Wazuh](images/architecture/soc-architecture/combining-elk-wazuh-hids-and-elastalert-for-optimal-performance.png)
![Wazuh Components](images/architecture/soc-architecture/wazuh-central-components.png)

### 📊 Wazuh Dashboard

![Dashboard](images/architecture/wazuh-dashboard/wazuh-dashboard-and-agent-deployment.png)
![Log Analysis](images/architecture/wazuh-dashboard/log-data-analysis.png)
![Cloud Security](images/architecture/wazuh-dashboard/monitoring-and-securing-cloud-workloads-with-wazuh.png)
![Microsoft Graph](images/architecture/wazuh-dashboard/monitoring-microsoft-graph-services-with-wazuh.png)

### 🚨 Brute Force Detection

![4625 Failed Logon](images/architecture/bruteforce-alert/event-4625-failed-to-logon.png)
![Detailed Logs](images/architecture/bruteforce-alert/failed-logon-events-id-4625-when-successfully-scanning-and-deploying-to-computers.png)
![Process Info](images/architecture/bruteforce-alert/process-information.png)
![Windows Event](images/architecture/bruteforce-alert/windows-event-id-4625-failed-logon.png)

### ⚔️ Attack Simulation

![Nmap Result](images/attacks/nmap-scan/interpreting-scan-results.png)
![Brute Force](images/attacks/nmap-scan/brute-force-attack.jpg)
![Terminal Scan](images/attacks/nmap-scan/nmap-scan-terminal-kali.webp)
![External Scan](images/attacks/nmap-scan/nmap-external-port-scan-result.webp)

### 🪟 Sysmon Logs

![Network](images/logs/sysmon-process/sysmon-logs-network.png)
![Event Log](images/logs/sysmon-process/generated-event-is-recorded-in-the-windows-event-log.png)
![RDP Issue](images/logs/sysmon-process/sysmon-event-id-3---rdp-logon-issue-initiated--field-allways-false.png)
![Workstation](images/logs/sysmon-process/workstation-logs.png)

---

## 💼 Skills Demonstrated

- Real SOC detection workflow (attack → alert → investigation → closure)
- SIEM deployment and custom rule authoring (Wazuh)
- Endpoint telemetry with Sysmon
- MITRE ATT&CK framework application
- Incident timeline reconstruction

---

## 👤 Author

**Sandeep Mothukuri**
- Website: [cybertechnology.in](https://cybertechnology.in)
- GitHub: [@sandeepmothukuri](https://github.com/sandeepmothukuri)
- LinkedIn: [sandeepmothukuris](https://www.linkedin.com/in/sandeepmothukuris)

---

## 🗂️ All Repositories

| Repository | Description |
|---|---|
| [ai-soc-lab](https://github.com/sandeepmothukuri/ai-soc-lab) | AI-augmented SOC with Wazuh + TheHive + Ollama (LLaMA3) for automated triage |
| [advanced-soc-forge](https://github.com/sandeepmothukuri/advanced-soc-forge) | 12-tool SOC lab with OpenSearch, Suricata, Zeek, MISP, Caldera, Velociraptor |
| [Autonomous-SOC-Lab](https://github.com/sandeepmothukuri/Autonomous-SOC-Lab) | Autonomous SOC with AI-driven detection and self-healing playbooks |
| [soc-threat-hunting-lab](https://github.com/sandeepmothukuri/soc-threat-hunting-lab) | Threat detection lab — Zeek, RITA, Arkime, Velociraptor, OSQuery, MISP |
| [soc-lab-free](https://github.com/sandeepmothukuri/soc-lab-free) | Free SOC lab — OpenVAS, Wazuh, pfSense, Proxmox Mail, Lynis |
| [soc-lab](https://github.com/sandeepmothukuri/soc-lab) | SOC analyst home lab — Wazuh SIEM, Sysmon, MITRE ATT&CK mapping |
| [cyberblue](https://github.com/sandeepmothukuri/cyberblue) | Containerised blue team platform — SIEM, DFIR, CTI, SOAR, Network Analysis |
