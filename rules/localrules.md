# local.rules – Custom Signatures Used in This Lab

This file contains the custom Suricata signatures used to validate detection capability in a controlled, segmented environment.

Lab Context:
Attacker (Kali): 192.168.100.10  
Suricata Gateway/IDS: 192.168.100.1 / 192.168.200.1  
Target (DVWA): 192.168.200.10  

All traffic from SOC-IN to SOC-OUT is routed through Suricata, ensuring the IDS observes both reconnaissance and web exploitation activity.

Rule Source:
These rules are loaded via suricata.yaml into Suricata at startup.

Proof that local rules are enabled in configuration:

![Local Rules Loaded in Config](../evidence/screenshots/02_Rule_Engine_Validation/01_local_rules_loaded_in_config.png)

Proof that rules load successfully:

![Rules Loaded Successfully](../evidence/screenshots/02_Rule_Engine_Validation/02_suricata_rules_loaded_successfully.png)

Proof of local rule contents as loaded on the Suricata VM:

![Custom Local Rules Content](../evidence/screenshots/02_Rule_Engine_Validation/05_custom_local_rules_content.png)

---

## 1) ICMP Validation Rule (Connectivity + Visibility)

Goal:
Confirm Suricata can see cross-subnet traffic and generate an alert for a simple L3 event.

Detection:
Any ICMP traffic (commonly triggered during ping-based path validation).

Rule:

```suricata
alert icmp any any -> any any (msg:"LAB TEST ALERT"; sid:1000001; rev:1;)
```

Why it works:
- ICMP is easy to generate and deterministic
- Confirms inline placement and packet visibility before moving to complex attack payloads

Validation evidence:

![Fastlog Alert Triggered Proof](../evidence/screenshots/02_Rule_Engine_Validation/03_fastlog_alert_triggered_proof.png)

---

## 2) SYN Scan Detection Rule (Recon – TCP Flags)

Goal:
Detect stealth port discovery activity typical of reconnaissance behavior.

Detection:
TCP SYN packets (flags:S), commonly produced by Nmap SYN scan (-sS).

Rule:

```suricata
alert tcp any any -> 192.168.200.10 any (flags:S; msg:"SYN SCAN DETECTED"; sid:1000003; rev:1;)
```

What it detects:
- Incomplete handshake behavior
- “Knocking” patterns against services
- Early-stage attacker recon

Validation evidence (scan alerts visible):

![Nmap Detection Alerts Fastlog](../evidence/screenshots/02_Rule_Engine_Validation/07_nmap_detection_alerts_fastlog.png)

---

## 3) SQL Injection Rule (Web Exploitation – Payload Keyword Match)

Goal:
Detect SQL injection attempts against DVWA by matching common SQLi keywords.

Detection:
The keyword "UNION" (case-insensitive) commonly appears in UNION-based SQL injection.

Rule:

```suricata
alert tcp any any -> 192.168.200.10 80 (msg:"RAW SQLI UNION DETECTED"; content:"UNION"; nocase; sid:1002001; rev:1;)
```

Why this rule exists:
- It is deterministic for lab validation
- It demonstrates content-based detection
- It proves Suricata can inspect payloads on HTTP traffic crossing the gateway

Notes:
- This is a simple signature (keyword match). In production, you would add:
  - http_uri/http_client_body keywords
  - flow:to_server,established
  - content anchoring, PCRE, and normalization checks
- For a lab portfolio, it cleanly proves payload inspection and alerting logic.

Runtime detection proof (alerts generated during exploit activity):

![Suricata Alert Generated Runtime](../evidence/screenshots/02_Rule_Engine_Validation/06_suricata_alert_generated_runtime.png)

---

## 4) XSS Detection Rule (Web Exploitation – Script Injection)

Goal:
Detect basic script injection attempts using common XSS markers.

Detection:
The tag "<script>" (case-insensitive).

Rule:

```suricata
alert tcp any any -> 192.168.200.10 80 (msg:"XSS ATTEMPT DETECTED"; content:"<script>"; nocase; sid:1003001; rev:1;)
```

Why it works:
- Script tags are a common and direct XSS indicator
- DVWA low-security mode makes this highly reproducible
- Confirms Suricata can detect client-supplied payload content

Operational notes:
- This rule matches the raw string. Real-world improvements often include:
  - URL decoding normalization
  - detecting encoded payloads (%3Cscript%3E)
  - restricting scope to HTTP URI/body fields for fewer false positives

---

## 5) SID Strategy (Avoiding Conflicts)

SIDs were selected to avoid collision with ET Open:

- 1000001 → ICMP visibility test
- 1000003 → SYN scan detection
- 1002001 → SQLi keyword detection
- 1003001 → XSS keyword detection

This makes alert triage easier and prevents rule ID overlap with community feeds.

---

## 6) Where Alerts Appear

fast.log (human-readable alerts):
/var/log/suricata/fast.log

eve.json (structured SOC/SIEM-ready output):
/var/log/suricata/eve.json

fast.log proof:

![Fastlog Alert Triggered Proof](../evidence/screenshots/02_Rule_Engine_Validation/03_fastlog_alert_triggered_proof.png)

ET Open rule base presence proof (baseline ruleset active alongside custom rules):

![ET Open Rule Base Statistics](../evidence/screenshots/02_Rule_Engine_Validation/04_rule_base_statistics_et_open.png)

---

## 7) Summary

These custom rules prove the following capabilities:

- L3 visibility and alerting (ICMP)
- L4 reconnaissance detection (TCP SYN scan)
- L7 payload inspection (SQLi + XSS keyword signatures)
- Successful integration alongside ET Open baseline rules
- Verified logging to fast.log and runtime alert generation

This completes the custom detection engineering layer of the Suricata IDS Detection Lab.
