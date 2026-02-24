# Validation Commands – Infrastructure & Detection Verification

This document outlines all commands used to validate:

- Network segmentation
- Routing enforcement
- Suricata configuration integrity
- Rule loading
- Runtime engine status
- Alert generation
- Packet inspection accuracy

These validation steps ensure the Suricata IDS Detection Lab is functioning correctly before and during attack simulation.

Lab Context:
Attacker (Kali): 192.168.100.10  
Suricata Gateway: 192.168.100.1 / 192.168.200.1  
Target (DVWA): 192.168.200.10  

---

## 1) Interface Verification (Suricata VM)

Before enabling detection, confirm both interfaces are active and correctly assigned.

Command:
ip -br a

![Interfaces Up](../evidence/screenshots/01_Infrastructure_Setup/03_suricata_interfaces_up.png)

Expected:
- enp0s3 → 192.168.100.1/24
- enp0s8 → 192.168.200.1/24
- Both interfaces UP

Failure symptoms:
- Missing IP
- DOWN state
- Incorrect subnet assignment

If incorrect, segmentation enforcement will fail.

---

## 2) IP Forwarding Validation

Suricata acts as a routing gateway. IPv4 forwarding must be enabled.

Command:
cat /proc/sys/net/ipv4/ip_forward

Expected output:
1

If disabled:
sudo sysctl -w net.ipv4.ip_forward=1

Proof of enabled forwarding:

![IP Forwarding Enabled](../evidence/screenshots/01_Infrastructure_Setup/04_ip_forwarding_enabled.png)

Without forwarding:
- Packets will not traverse between SOC-IN and SOC-OUT
- Attacks will fail
- IDS inspection will not see routed traffic

---

## 3) Suricata Configuration Validation (Test Mode)

Before running Suricata in live capture mode, validate YAML configuration.

Command:
sudo suricata -T -c /etc/suricata/suricata.yaml

![Configuration Validation Success](../evidence/screenshots/01_Infrastructure_Setup/05_configuration_validation_success.png)

Expected:
- Configuration successfully loaded
- Rule files processed
- No syntax errors

If errors appear:
- Fix rule path
- Correct YAML indentation
- Ensure rule files exist

---

## 4) Runtime Engine Initialization Check

Start Suricata in detection mode and verify engine startup.

Command:
sudo suricata -c /etc/suricata/suricata.yaml -i enp0s3 -v

Engine initialization proof:
![Engine Initialization](../evidence/screenshots/01_Infrastructure_Setup/06_suricata_engine_initialized.png)

Thread creation proof:
![Runtime Threads](../evidence/screenshots/01_Infrastructure_Setup/07_suricata_runtime_threads.png)

Validate:
- Engine started
- Flow manager initialized
- Capture threads created
- No fatal errors

If threads are not created, detection will not function.

---

## 5) Rule Loading Verification

Ensure both ET Open and custom local rules are loaded.

Check rule file inclusion:
![Local Rules Included](../evidence/screenshots/02_Rule_Engine_Validation/01_local_rules_loaded_in_config.png)

Check rule load status:
![Rules Loaded Successfully](../evidence/screenshots/02_Rule_Engine_Validation/02_suricata_rules_loaded_successfully.png)

Check baseline rule statistics:

![ET Open Rule Statistics](../evidence/screenshots/02_Rule_Engine_Validation/04_rule_base_statistics_et_open.png)

Expected:
- Rule files processed > 0
- Rules successfully loaded
- 0 rules failed
- High signature count (ET baseline active)

If rule count is low:
- Rule path misconfigured
- Suricata rules not downloaded
- YAML misconfiguration

---

## 6) Custom Local Rule Verification

Confirm custom detection signatures are present.

Command:
cat /var/lib/suricata/rules/local.rules

Proof of rule content:
![Custom Local Rules](../evidence/screenshots/02_Rule_Engine_Validation/05_custom_local_rules_content.png)

Expected to see:
- ICMP test alert
- SYN scan detection rule
- SQLi detection rule
- XSS detection rule

If missing:
- Local rule file not loaded
- Incorrect rule path in suricata.yaml

---

## 7) Alert Generation Validation (fast.log)

After generating traffic (ping, scan, or HTTP payload), verify alert output.

Command:
sudo tail -f /var/log/suricata/fast.log

Proof of alert generation:
![Fastlog Alert Proof](../evidence/screenshots/02_Rule_Engine_Validation/03_fastlog_alert_triggered_proof.png)

Expected:
- Alert message
- Signature ID
- Source IP
- Destination IP
- Port

Example detections:
- LAB TEST ALERT
- SYN SCAN DETECTED
- RAW SQL UNION DETECTED
- XSS ATTEMPT DETECTED

If no alerts appear:
- Rule not triggered
- Incorrect HOME_NET configuration
- Suricata not capturing correct interface

---

## 8) JSON Alert Validation (eve.json)

Structured alerts must also be validated.

Command:
sudo tail -n 50 /var/log/suricata/eve.json | grep '"event_type":"alert"'

Expected fields:
- signature
- signature_id
- src_ip
- dest_ip
- http.uri (for web attacks)
- protocol
- severity

This confirms:
- App-layer parsing active
- HTTP decoding functioning
- Structured SOC-style output available

---

## 9) Traffic Path Validation (Packet Traversal)

To confirm Suricata sits inline between attacker and victim:

Monitor ingress:
sudo tcpdump -i enp0s3

Monitor egress:
sudo tcpdump -i enp0s8

When Kali generates traffic:
- Packet appears first on enp0s3
- Then forwarded and visible on enp0s8

This confirms:
- Routing active
- Segmentation enforced
- IDS positioned correctly in traffic path

---

## 10) Recon Detection Validation

After running an Nmap scan, confirm detection.

Proof:
![Nmap Detection Alerts](../evidence/screenshots/02_Rule_Engine_Validation/07_nmap_detection_alerts_fastlog.png)

Expected alerts:
- SYN SCAN DETECTED
- ET SCAN Possible Nmap User-Agent Observed

This confirms reconnaissance detection capability.

---

## 11) Web Exploitation Detection Validation

After SQLi or XSS payload execution:

Verify fast.log:
Alert triggered

Verify eve.json:
HTTP fields present (URI, method, payload)

This confirms:
- Layer 7 inspection
- HTTP payload decoding
- Signature match success

---

## Validation Checklist

Before declaring the lab operational, confirm:

- Interfaces UP
- Correct IP addressing
- IP forwarding enabled
- suricata.yaml validated
- Rules successfully loaded
- Suricata engine running
- fast.log generating alerts
- eve.json logging structured events
- Traffic visible on both interfaces

---

## Summary

These validation commands ensure:

- Network segmentation is enforced
- Suricata is correctly positioned inline
- Routing functions properly
- Detection engine is operational
- Rule base is active
- Alerts are generated and logged
- Application-layer inspection is working

This validation process guarantees that all reconnaissance and exploitation scenarios are inspected, detected, and reproducible within the Suricata IDS Detection Lab.
