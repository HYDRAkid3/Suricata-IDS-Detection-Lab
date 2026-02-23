# 04 - SQL Injection Case Study

## Objective

Simulate SQL Injection attack against DVWA and validate Suricata detection.

---

## Attack Execution

Access DVWA:
http://192.168.200.10

Payload example:
' OR 1=1 --

---

## Custom Rule

Example rule:

```
alert http any any -> any any (msg:"SQL Injection Attempt"; content:"' OR 1=1"; sid:1000002; rev:1;)
```

---

## Log Validation

```bash
sudo tail -n 5 /var/log/suricata/fast.log
sudo tail -n 5 /var/log/suricata/eve.json
```

---

## Evidence

![SQLi Payload](../assets/screenshots/sqli/01-payload.png)

![fast.log SQLi](../assets/screenshots/sqli/02-fastlog-sqli.png)

![eve.json SQLi](../assets/screenshots/sqli/03-evejson-sqli.png)

---

## MITRE ATT&CK

T1190 – Exploit Public-Facing Application

---

## Findings

- Payload successfully triggered detection
- HTTP inspection functioning
- Structured alert logged in eve.json
