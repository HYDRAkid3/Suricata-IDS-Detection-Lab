# 06 - Reconnaissance Scan

## Objective

Simulate network scanning using Nmap and validate detection.

---

## Command Used

```bash
nmap -sS 192.168.200.10
```

---

## Log Validation

```bash
sudo tail -n 5 /var/log/suricata/fast.log
```

---

## Evidence

![Nmap Scan](../assets/screenshots/recon/01-nmap-scan.png)

![fast.log Recon](../assets/screenshots/recon/02-fastlog-recon.png)

---

## MITRE ATT&CK

T1046 – Network Service Scanning

---

## Findings

- SYN scan detected
- Suricata flagged reconnaissance behavior
- IDS monitoring validated
