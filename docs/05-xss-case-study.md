# 05 - Cross-Site Scripting (XSS) Case Study

## Objective

Simulate reflected XSS attack and validate detection via Suricata.

---

## Attack Payload

<script>alert(1)</script>

---

## Custom Rule

```
alert http any any -> any any (msg:"XSS Attempt"; content:"<script>"; sid:1000003; rev:1;)
```

---

## Log Validation

```bash
sudo tail -n 5 /var/log/suricata/fast.log
```

---

## Evidence

![XSS Payload](../assets/screenshots/xss/01-xss-payload.png)

![fast.log XSS](../assets/screenshots/xss/02-fastlog-xss.png)

---

## MITRE ATT&CK

T1059 – Command and Scripting Interpreter

---

## Findings

- HTTP payload inspection successful
- Alert generated in real-time
- Detection confirmed via fast.log
