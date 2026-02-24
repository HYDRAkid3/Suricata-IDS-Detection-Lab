# Network Architecture (Dual-Network Suricata IDS Lab)

## Goal (What this architecture proves)

This lab is built to **prove end-to-end traffic inspection** using Suricata in a segmented network design:

- Kali (attacker) cannot directly reach the DVWA server without traffic passing through Suricata
- Suricata inspects packets **in transit** (inline routing/forwarding path)
- Alerts are validated using both:
  - `fast.log` (quick human-readable alerts)
  - `eve.json` (structured JSON events for analysis)

In short: **Attack → Transit via Suricata → Detect → Validate logs → Evidence**

---

## Architecture Diagram

> If your screenshot filename is different, just update the path below.

![Suricata Dual Network Architecture](../evidence/screenshots/00_Architecture/01_suricata_ids_dual_network_architecture.png)

---

## Lab Components (Who does what)

### 1) Kali Linux (Attacker)
Kali generates the traffic that should trigger Suricata detections:
- Reconnaissance scans (Nmap)
- Connectivity checks (ICMP ping / basic TCP checks)
- Web exploitation attempts against DVWA:
  - SQL Injection
  - XSS

**Purpose:** create controlled attacker behavior and payloads.

---

### 2) Suricata IDS (Inspection / Transit Node)
Suricata is the **inspection point** in the middle:
- Has **two network interfaces** (one facing Kali side, one facing Ubuntu side)
- Captures traffic and evaluates it against:
  - ET Open rules (baseline coverage)
  - Custom rules (your own detection logic)
- Forwards traffic between networks using IP forwarding (routing)

**Purpose:** simulate a real inline inspection point and generate detection evidence.

Key outputs used in this project:
- `/var/log/suricata/fast.log`
- `/var/log/suricata/eve.json`

---

### 3) Ubuntu (Victim / DVWA Server)
Ubuntu hosts DVWA and receives traffic only through Suricata:
- DVWA web app is the target
- Receives HTTP requests and exploitation payloads
- Used to verify that the attack traffic actually reached the service

**Purpose:** controlled victim for web exploitation + proof of traffic transit.

---

## Network Segmentation (Two isolated networks)

This lab uses **two separate internal networks** to force traffic through Suricata:

### SOC-IN (Ingress Side)
- Network where Kali lives
- Suricata’s **ingress interface** connects here
- All attacker traffic enters Suricata from this side

### SOC-OUT (Egress Side)
- Network where Ubuntu (DVWA) lives
- Suricata’s **egress interface** connects here
- Suricata forwards inspected traffic out to Ubuntu here

**Why two networks?**
Because it prevents accidental “direct routing” and ensures Suricata stays the chokepoint.

---

## Traffic Flow (How packets move)

### High-Level Flow
Kali (Attacker) ---> Suricata (Inspection/Forwarding) ---> Ubuntu (DVWA)


### Step-by-Step Flow (What actually happens)
1. Kali sends traffic (ping / scan / HTTP exploit).
2. Packet reaches Suricata on **SOC-IN interface**.
3. Suricata inspects:
   - packet headers (IP/port/protocol)
   - payload content (HTTP content patterns, signatures)
4. If traffic matches a rule:
   - Suricata generates an alert in `fast.log`
   - Suricata records full event context in `eve.json`
5. Suricata forwards the traffic out via **SOC-OUT interface**.
6. Ubuntu receives the traffic (DVWA request reaches server).

This guarantees:
- **Visibility:** Suricata sees the traffic before the server does  
- **Evidence:** alerts + logs prove inspection + detection  
- **Control:** traffic path is deterministic and repeatable

---

## Why IP Forwarding Matters (Critical design detail)

Suricata is not just “sniffing” random traffic.  
It is positioned in the network path so packets **must pass through it**.

To achieve this, Suricata host enables:
- IP forwarding (acts like a router between SOC-IN and SOC-OUT)

This is what makes the architecture realistic:  
**it resembles a true inline inspection deployment** rather than a passive mirror port.

---

## Detection Workflow (How alerts are generated)

This lab tests detection in a structured way:

1. **Baseline detection enabled**
   - ET Open rules provide broad coverage
2. **Custom rules added**
   - Specific signatures for your lab scenarios
3. **Attack simulation performed**
   - Recon scans and web exploit payloads
4. **Detection validated**
   - `fast.log` proves alert firing
   - `eve.json` proves structured fields and metadata

---

## Logging & Evidence (What you collect and why)

### fast.log (Quick proof)
Used for:
- “Did the alert fire?”
- Quick screenshot evidence for GitHub

### eve.json (Structured analysis)
Used for:
- Timestamp, src/dst IPs, ports
- Signature ID (SID), category, severity
- HTTP fields and payload context (when available)

This is the log format used in real SOC pipelines because it’s machine-readable.

---

## What This Architecture Demonstrates (Skills)

This design is not just “running Suricata.” It demonstrates:

- Network segmentation and traffic control
- Inline inspection placement (routing/forwarding)
- Signature-based detection engineering concepts
- Rule validation and alert verification
- Evidence-driven incident investigation workflow
- Practical SOC-style documentation and reporting

---

## How This Maps to the Repo (Where proof is stored)

- Architecture screenshots:
  - `evidence/screenshots/00_Architecture/`
- Infrastructure + interface proof:
  - `evidence/screenshots/01_Infrastructure_Setup/`
- Rule engine proof:
  - `evidence/screenshots/02_Rule_Engine_Validation/`
- Network path proof:
  - `evidence/screenshots/03_Network_Path_Validation/`
- Recon proof:
  - `evidence/screenshots/04_Recon_Scan/`
- Web exploitation proof:
  - `evidence/screenshots/05_SQLi_Case_Study/`
  - `evidence/screenshots/06_XSS_Case_Study/`

---

## Final Summary

This dual-network architecture ensures Suricata is the inspection chokepoint between attacker and victim.  
It produces repeatable detection results and strong evidence through both packet-level validation and alert log analysis.
