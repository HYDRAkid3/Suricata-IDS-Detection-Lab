# Suricata IDS Detection Lab  
Network Segmentation | Detection Engineering | Attack Simulation

---

## Overview

This project demonstrates the deployment of **Suricata 7 IDS as a routing inspection gateway** in a segmented lab environment.

The lab simulates:

- An attacker performing reconnaissance and exploitation
- A vulnerable web server (DVWA)
- A network-based detection engine inspecting all traffic between segments

The objective is to validate detection logic using:

- ET Open baseline signatures
- Custom Suricata rules
- Packet-level inspection
- Structured event analysis
- MITRE ATT&CK mapping

---

## Threat Model

This lab models a segmented internal network where:

- An attacker machine (Kali Linux) attempts reconnaissance and exploitation.
- A vulnerable web application (DVWA) resides in a separate subnet.
- Suricata operates as an inline routing gateway between segments.

Attack surface:
- ICMP (path validation)
- TCP SYN scans
- HTTP (SQL Injection / XSS payloads)

Detection objective:
Identify reconnaissance and exploitation attempts at the network boundary before they reach the application layer.

---

## Architecture

Suricata operates with dual network interfaces:

- SOC-IN  → 192.168.100.0/24  
- SOC-OUT → 192.168.200.0/24  

Traffic path enforced:

Kali → Suricata (Inspection) → DVWA

### Architecture Diagram

![Suricata Architecture](assets/diagrams/Suricata%20IDS%20Architecture%20Diagram.png)

Key components:

- Kali Linux (Attacker)
- Suricata IDS VM (Dual NIC Routing Gateway)
- Ubuntu DVWA Web Server
- Packet capture validation via tcpdump
- Alert output via fast.log and eve.json

---

## Detection Engineering Approach

The detection strategy follows a layered model:

1. **ET Open baseline rule set**  
   ~48,000+ community signatures provide broad threat detection.

2. **Custom lab rules (local.rules)**  
   Deterministic detection for:
   - ICMP validation
   - SYN scan detection (threshold-based)
   - SQL Injection indicators
   - XSS indicators

3. **Packet-level validation**
   tcpdump used to confirm:
   - Traffic traversal
   - Payload visibility
   - Detection accuracy

4. **Structured event logging**
   Suricata outputs:
   - fast.log (human-readable alerts)
   - eve.json (structured JSON for SIEM ingestion)

This mirrors real SOC environments where community signatures are supplemented with environment-specific detection logic.

---

## Simulated Attack Scenarios

### 1. Network Reconnaissance
- SYN scan detection
- ICMP path validation

### 2. SQL Injection
Payload example:
```
' OR 1=1 --
```

Detection logic:
- URI content matching
- HTTP inspection

MITRE Mapping:
- T1190 – Exploit Public-Facing Application

---

### 3. Cross-Site Scripting (XSS)
Payload example:
```
<script>alert(1)</script>
```

Detection logic:
- Script tag inspection
- Encoded payload detection

MITRE Mapping:
- T1059 – Command and Scripting Interpreter

---

## Evidence & Validation

The project includes:

- Rule engine validation screenshots
- ET Open rule count verification (~48k rules)
- Custom rule file proof
- Alert validation via fast.log
- Packet capture validation using tcpdump
- Architecture documentation

Each case study in the `docs/` directory includes step-by-step analysis and screenshots.

---

## MITRE ATT&CK Mapping

| Technique | ID | Scenario |
|-----------|----|----------|
| Network Service Scanning | T1046 | SYN scan |
| Exploit Public-Facing Application | T1190 | SQLi |
| Command & Scripting Injection | T1059 | XSS |

This mapping demonstrates detection alignment with standardized threat frameworks.

---

## Limitations

- DVWA is intentionally vulnerable and does not reflect hardened production systems.
- HTTPS traffic decryption was not implemented.
- Detection tuning was minimal due to controlled lab environment.
- No host-based telemetry was integrated.

---

## Future Enhancements

- Integrate Suricata with Wazuh or SIEM platform
- Enable TLS inspection testing
- Deploy Suricata in IPS mode
- Implement rule tuning & false positive reduction
- Add automated alert enrichment pipeline
- Introduce performance benchmarking

---

## What This Project Demonstrates

- Network segmentation enforcement
- IDS deployment in routing mode
- Community signature integration (ET Open)
- Custom detection rule development
- Wire-level packet validation
- Structured event analysis
- MITRE ATT&CK alignment
- Detection engineering mindset

---

## Author

Harshit Krishna  
MS Cybersecurity - University of Delaware  
Blue Team / SOC / Detection Engineering Focus
