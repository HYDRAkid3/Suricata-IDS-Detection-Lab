# 04 - SQL Injection Case Study

## Objective

Simulate SQL Injection attack against DVWA and validate Suricata detection.

---

## DVWA SQLi Execution

![DVWA SQLi](../assets/screenshots/4%20SQL%20Injection%20Case%20Study/01_dvwa_sqli_success.png)

SQL injection payload successfully executed.

---

## tcpdump Payload Capture

![SQLi Packet](../assets/screenshots/4%20SQL%20Injection%20Case%20Study/02_tcpdump_sqli_payload.png)

Payload visible at packet inspection level.

---

## Suricata Alert (fast.log)

![fast.log SQLi](../assets/screenshots/4%20SQL%20Injection%20Case%20Study/03_fast_log_sqli_alert.png)

Detection triggered by custom rule.

---

## Application Response

![App Response](../assets/screenshots/4%20SQL%20Injection%20Case%20Study/04_application_response.png)

Server response confirms successful injection attempt.

---

## MITRE ATT&CK

T1190 – Exploit Public-Facing Application

---

## Findings

- HTTP payload inspection operational
- SQLi rule successfully triggered
- Alert logged in fast.log
