# Detection Engine – ET Open Baseline & Rule Validation

This document describes the Suricata detection engine configuration, baseline rule set (ET Open), custom rule integration, and runtime alert validation within the segmented lab environment.

Lab Context:
Attacker (Kali): 192.168.100.10  
Suricata Gateway: 192.168.100.1 / 192.168.200.1  
Target (DVWA): 192.168.200.10  

The objective of this section is to prove:

- Suricata rule engine is operational
- ET Open baseline rules are loaded
- Custom rules are correctly integrated
- Alerts trigger for reconnaissance and web attacks
- Structured logging is functional

---

## 1) Rule Configuration in suricata.yaml

Suricata must reference rule files inside the configuration file.

Proof that local rules are included in the configuration:

![Local Rules Loaded in Config](../evidence/screenshots/02_Rule_Engine_Validation/01_local_rules_loaded_in_config.png)

This confirms:
- Custom rule file path correctly defined
- Suricata will process local.rules at startup

If this section is missing, custom detections will never trigger.

---

## 2) Rule Loading Validation

After configuration validation, rule loading is verified at startup.

Command:
sudo suricata -T -c /etc/suricata/suricata.yaml

Proof of successful rule loading:

![Rules Loaded Successfully](../evidence/screenshots/02_Rule_Engine_Validation/02_suricata_rules_loaded_successfully.png)

Expected:
- Rule files processed > 0
- Rules successfully loaded
- 0 rules failed
- High signature count

If rules fail to load:
- Incorrect rule path
- Syntax error in local rule
- YAML indentation issue

---

## 3) ET Open Baseline Rule Statistics

ET Open provides the baseline detection coverage.

Proof of active rule base:

![ET Open Rule Statistics](../evidence/screenshots/02_Rule_Engine_Validation/04_rule_base_statistics_et_open.png)

Observations:
- Large number of rules loaded
- Categories include:
  - SCAN
  - WEB_ATTACK
  - POLICY
  - EXPLOIT
- Engine reports rule parsing success

This ensures:
- Recon detection coverage
- Web exploitation coverage
- Known threat pattern detection

---

## 4) Custom Local Rule Verification

Custom rules are used to simulate lab-specific detections.

Command:
cat /var/lib/suricata/rules/local.rules

Proof:

![Custom Local Rules Content](../evidence/screenshots/02_Rule_Engine_Validation/05_custom_local_rules_content.png)

Custom detections include:

- LAB TEST ALERT
- SYN SCAN DETECTED
- RAW SQL UNION DETECTED
- XSS ATTEMPT DETECTED

Each rule contains:
- msg field (alert description)
- sid (unique signature ID)
- rev (revision number)
- content match (payload trigger)

Custom rules validate:
- Signature matching logic
- Payload inspection capability
- Rule-writing competency

---

## 5) Runtime Alert Generation

During attack simulation, Suricata generates alerts in real time.

Proof of runtime alert generation:

![Runtime Alert Generation](../evidence/screenshots/02_Rule_Engine_Validation/06_suricata_alert_generated_runtime.png)

This confirms:
- Detection engine active
- Traffic reaching rule evaluation stage
- Alerts written during live packet processing

---

## 6) Reconnaissance Detection (Nmap)

During SYN scan activity from 192.168.100.10, alerts are generated.

Proof:

![Nmap Detection Alerts](../evidence/screenshots/02_Rule_Engine_Validation/07_nmap_detection_alerts_fastlog.png)

Observed detections:
- SYN SCAN DETECTED
- ET SCAN Possible Nmap User-Agent Observed

Detection logic:
- TCP flag pattern recognition
- User-Agent pattern matching
- Scan rate behavior

This confirms:
- L4 inspection functional
- Scan behavior identifiable
- ET Open reconnaissance signatures operational

---

## 7) SQL Injection Detection

Attack:
Boolean-based injection against DVWA

Detection proof in fast.log:

![SQLi Fastlog Detection](../evidence/screenshots/05_SQLi_Case_Study/03_suricata_fastlog_multi_attack_detection.png)

Detection proof in eve.json:

![SQLi eve.json Alert](../evidence/screenshots/05_SQLi_Case_Study/04_suricata_evejson_raw_sqli_union_alert.png)

Key fields:
- src_ip: 192.168.100.10
- dest_ip: 192.168.200.10
- http.uri: /DVWA/vulnerabilities/sqli/
- signature_id: 1002001
- signature: RAW SQL UNION DETECTED

This confirms:
- Layer 7 HTTP parsing active
- Payload inspection functional
- Custom SQLi rule correctly triggered

---

## 8) XSS Detection

Attack:
<script>alert(1)</script>

Packet-level visibility proof:

![XSS Payload Capture](../evidence/screenshots/06_XSS_Case_Study/01_http_xss_payload_packet_capture.png)

Fast.log alert:

![XSS Fastlog Alert](../evidence/screenshots/06_XSS_Case_Study/02_fastlog_xss_alert_triggered.png)

Structured JSON alert:

![XSS eve.json Details](../evidence/screenshots/06_XSS_Case_Study/03_evejson_xss_alert_details.png)

Observed:
- http.method: GET
- http.uri includes script payload
- signature_id: 1003001
- direction: to_server

This confirms:
- Application-layer inspection operational
- HTTP decoding working
- Payload signature detection successful

---

## 9) Detection Coverage Summary

| Attack Type | Detection Source | Log Type | Status |
|-------------|------------------|----------|--------|
| ICMP Test   | Custom Rule      | fast.log | Detected |
| SYN Scan    | ET + Custom      | fast.log | Detected |
| SQLi        | Custom Rule      | fast.log + eve.json | Detected |
| XSS         | Custom Rule      | fast.log + eve.json | Detected |

---

## 10) Detection Architecture Strength

The detection engine demonstrates:

- Multi-interface packet capture
- Layer 3/4 signature matching
- Layer 7 HTTP payload inspection
- Structured alert generation
- Integration of community rules (ET Open)
- Custom signature authoring capability

The IDS operates in detection mode (alert-only), preserving lab traffic flow while providing full observability.

---

## 11) Conclusion

The Suricata detection engine is fully operational and validated.

This section confirms:

- Rule base loaded successfully
- Custom rules integrated
- Reconnaissance detection working
- Web exploitation detection working
- Structured logging enabled
- Inline inspection architecture functioning

All simulated adversarial activity is successfully detected and logged within the Suricata IDS Detection Lab.
