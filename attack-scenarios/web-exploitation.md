# Web Exploitation Scenarios – SQL Injection & XSS Detection

This section simulates application-layer attacks against DVWA hosted on:

Target: 192.168.200.10  
Attacker: 192.168.100.10  
Inspection Node: Suricata (192.168.100.1 / 192.168.200.1)

All traffic is inspected inline before reaching the web server.

---

# Environment Validation

## DVWA Security Level Set to Low

![DVWA Security Level](../evidence/screenshots/05_SQLi_Case_Study/00_dvwa_security_level_configured_low.png)

Security level is intentionally set to **Low** to allow exploitation.

---

## Web Server Reachability

![Web Server Reachable](../evidence/screenshots/05_SQLi_Case_Study/01_dvwa_webserver_reachable.png)

Confirms:
- Apache is running
- DVWA accessible
- Target = 192.168.200.10
- Traffic path working

---

# SQL Injection Attack

## Attack Vector

Boolean-based injection: 1' OR '1'='1
Target URL: http://192.168.200.10/DVWA/vulnerabilities/sqli/


---

## Exploitation Success

![SQLi Boolean Bypass](../evidence/screenshots/05_SQLi_Case_Study/02_dvwa_sqli_boolean_bypass_success.png)

The injection returns multiple database records, confirming vulnerability.

This validates:
- Application-level injection
- HTTP payload delivered successfully
- Inline inspection occurred

---

## Suricata Detection – fast.log

![Fastlog Multi Attack](../evidence/screenshots/05_SQLi_Case_Study/03_suricata_fastlog_multi_attack_detection.png)

We observe:

- SYN SCAN DETECTED
- RAW SQL UNION DETECTED
- XSS ATTEMPT DETECTED (during later testing)

Key fields:
- src_ip: 192.168.100.10
- dest_ip: 192.168.200.10
- dest_port: 80

This confirms Suricata detected SQL injection payload.

---

## Suricata Detection – eve.json

![Eve.json SQLi Alert](../evidence/screenshots/05_SQLi_Case_Study/04_suricata_evejson_raw_sqli_union_alert.png)

Structured alert includes:

- signature: RAW SQL UNION DETECTED
- signature_id: 1002001
- http.uri: /DVWA/vulnerabilities/sqli/?id=...
- protocol: HTTP
- direction: to_server
- severity: 3

This confirms:
- Application-layer detection
- HTTP payload parsing successful
- Proper rule triggering

---

# XSS Attack

## Security Level Validation

![DVWA Security Level Low](../evidence/screenshots/06_XSS_Case_Study/00_dvwa_security_level_low.png)

Confirms DVWA is still vulnerable.

---

## XSS Payload

Injected payload: <script>alert(1)</script>
Target: http://192.168.200.10/DVWA/vulnerabilities/xss_r/


---

## Packet-Level Proof (Inline Inspection)

![HTTP XSS Payload Capture](../evidence/screenshots/06_XSS_Case_Study/01_http_xss_payload_packet_capture.png)

Captured on Suricata ingress interface:

- Source: 192.168.100.10
- Destination: 192.168.200.10
- HTTP GET request
- Payload visible in transit

This confirms:
- Suricata sees raw HTTP payload
- Inspection occurs before server response

---

## fast.log Detection

![Fastlog XSS Alert](../evidence/screenshots/06_XSS_Case_Study/02_fastlog_xss_alert_triggered.png)

Alert details:

- Signature: XSS ATTEMPT DETECTED
- Protocol: TCP
- Port: 80
- Source IP: 192.168.100.10
- Destination IP: 192.168.200.10

---

## eve.json Structured Alert

![Eve.json XSS Details](../evidence/screenshots/06_XSS_Case_Study/03_evejson_xss_alert_details.png)

Key fields:

- signature_id: 1003001
- signature: XSS ATTEMPT DETECTED
- http.method: GET
- http.uri: /DVWA/vulnerabilities/xss_r/?name=<script>...
- direction: to_server
- severity: 3

This confirms:

- Application-layer inspection
- Payload decoding successful
- Signature-based detection working
- Structured SOC-ready logging

---

# MITRE ATT&CK Mapping

## SQL Injection
- T1190 – Exploit Public-Facing Application

## Cross-Site Scripting
- T1059 – Command Execution via Web Input
- T1203 – Exploitation for Client Execution

---

# Detection Summary

| Attack | Detected in fast.log | Detected in eve.json | Inline Payload Visible |
|--------|---------------------|----------------------|------------------------|
| SQLi   | Yes                 | Yes                  | Yes                    |
| XSS    | Yes                 | Yes                  | Yes                    |

---

# Key Validation Points

- Suricata correctly positioned inline
- Traffic flows: 192.168.100.10 → 192.168.200.10
- Custom rules loaded and active
- ET Open baseline active
- Layer 7 HTTP inspection operational
- JSON logging functional

---

# Conclusion

The exploitation phase confirms:

- Recon detection works
- Web exploitation detection works
- Custom signatures fire correctly
- Structured alerts include full HTTP context
- Inline IDS architecture functions as designed

This demonstrates end-to-end detection validation from attack generation to structured alert analysis.


