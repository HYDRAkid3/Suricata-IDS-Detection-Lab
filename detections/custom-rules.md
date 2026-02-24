# Custom Detection Rules – Design, Logic & Validation

This document explains the custom Suricata detection rules written specifically for this lab. These rules were created to simulate real detection engineering workflows and validate packet inspection across network and application layers.

Lab Context:
Attacker (Kali): 192.168.100.10  
Suricata Gateway: 192.168.100.1 / 192.168.200.1  
Target (DVWA): 192.168.200.10  

All traffic traverses Suricata inline before reaching the victim server.

---

## 1) Local Rule File Verification

Custom rules are stored in:

/var/lib/suricata/rules/local.rules

Proof of rule content:

![Custom Local Rules](../evidence/screenshots/02_Rule_Engine_Validation/05_custom_local_rules_content.png)

These rules were loaded successfully during Suricata initialization.

---

## 2) Rule Structure Overview

Each Suricata rule follows this format:

alert <protocol> <source> <source_port> -> <destination> <destination_port> (rule options)

Core components:

- action: alert
- protocol: icmp / tcp / http
- source/destination matching
- content matching (payload inspection)
- msg (alert description)
- sid (unique signature ID)
- rev (revision number)

Example generic structure:

alert tcp any any -> any 80 (msg:"Example Alert"; content:"test"; sid:1000001; rev:1;)

---

## 3) ICMP Test Rule (Layer 3 Validation)

Purpose:
Verify that Suricata detects basic ICMP traffic between subnets.

Detection Type:
Layer 3 (ICMP protocol)

Trigger Condition:
Any ICMP echo request from attacker to victim.

Validation:
When Kali executes:

ping 192.168.200.10

An alert is generated in fast.log.

Proof of alert generation:

![Fastlog Alert Proof](../evidence/screenshots/02_Rule_Engine_Validation/03_fastlog_alert_triggered_proof.png)

This confirms:
- Traffic is visible to Suricata
- Rule engine evaluates non-TCP traffic
- Basic protocol detection operational

---

## 4) SYN Scan Detection Rule (Layer 4 – TCP Flags)

Purpose:
Detect stealth reconnaissance behavior using TCP SYN scanning.

Detection Type:
Layer 4 (TCP flag analysis)

Trigger Condition:
Packets with SYN flag set without ACK.

Example scan:

nmap -sS 192.168.200.10

Detection proof:

![Nmap Detection Alerts](../evidence/screenshots/02_Rule_Engine_Validation/07_nmap_detection_alerts_fastlog.png)

This confirms:
- TCP header inspection working
- Flag-based detection functional
- Recon activity identifiable at transport layer

Detection logic focuses on:
- TCP flag combinations
- Destination port targeting
- Repeated connection attempts

---

## 5) SQL Injection Detection Rule (Layer 7 – HTTP Payload Inspection)

Purpose:
Detect SQL injection payloads sent to DVWA.

Attack Example:
1' OR '1'='1
UNION SELECT ...

Target:
http://192.168.200.10/DVWA/vulnerabilities/sqli/

Detection Type:
Layer 7 (HTTP content matching)

The rule searches for SQL-specific keywords such as:
- UNION
- SELECT
- OR 1=1

Fast.log detection proof:

![SQLi Fastlog Detection](../evidence/screenshots/05_SQLi_Case_Study/03_suricata_fastlog_multi_attack_detection.png)

Structured JSON alert proof:

![SQLi eve.json Alert](../evidence/screenshots/05_SQLi_Case_Study/04_suricata_evejson_raw_sqli_union_alert.png)

Key fields in eve.json:
- src_ip: 192.168.100.10
- dest_ip: 192.168.200.10
- http.uri includes injection string
- signature_id: 1002001
- direction: to_server

This confirms:
- HTTP decoding active
- Payload inspected before application response
- Signature-based SQL injection detection working

---

## 6) XSS Detection Rule (Layer 7 – Script Injection)

Purpose:
Detect cross-site scripting attempts.

Attack Payload:
<script>alert(1)</script>

Target:
http://192.168.200.10/DVWA/vulnerabilities/xss_r/

Detection Type:
Layer 7 (HTTP content matching)

The rule searches for:
- "<script>"
- "alert("

Packet-level visibility proof:

![XSS Payload Capture](../evidence/screenshots/06_XSS_Case_Study/01_http_xss_payload_packet_capture.png)

Fast.log detection proof:

![XSS Fastlog Alert](../evidence/screenshots/06_XSS_Case_Study/02_fastlog_xss_alert_triggered.png)

Structured JSON detection proof:

![XSS eve.json Details](../evidence/screenshots/06_XSS_Case_Study/03_evejson_xss_alert_details.png)

Observed:
- HTTP GET request captured
- Script payload visible in URI
- signature_id: 1003001
- severity: 3

This confirms:
- Layer 7 inspection working
- HTTP payload parsing accurate
- Application-layer attack detection operational

---

## 7) Signature ID Strategy

Custom rule SIDs follow a structured numbering scheme:

- 1000001 → ICMP test
- 1001001 → SYN scan detection
- 1002001 → SQL injection detection
- 1003001 → XSS detection

Benefits:
- Avoid conflicts with ET Open rules
- Organized rule tracking
- Easier debugging and alert correlation

---

## 8) Detection Layer Mapping

| Rule Type | OSI Layer | Detection Focus |
|------------|-----------|-----------------|
| ICMP Test | Layer 3 | Protocol visibility |
| SYN Scan | Layer 4 | TCP flags & scan behavior |
| SQLi | Layer 7 | HTTP content inspection |
| XSS | Layer 7 | Script payload detection |

This demonstrates multi-layer inspection capability.

---

## 9) Operational Validation

During runtime:

- Suricata threads active
- Rules loaded successfully
- Alerts written to fast.log
- Structured events written to eve.json
- Traffic visible on both interfaces

This confirms:
- Inline inspection model working
- Detection engine functional
- Custom rule authoring validated

---

## 10) Conclusion

The custom rule set demonstrates:

- Practical detection engineering skills
- Payload-based signature development
- Multi-layer inspection validation
- Structured alert analysis capability
- Integration with ET Open baseline rules

All simulated attacks were successfully detected using custom signatures within the Suricata IDS Detection Lab.
