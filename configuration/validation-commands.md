# Suricata Validation Commands — Operational Workflow

This document outlines the validation commands used to confirm:

- Proper network configuration
- Traffic traversal through the IDS gateway
- Successful rule loading
- Alert generation
- Structured log output

The objective is to demonstrate operational discipline and systematic validation.

---

## 1. Network Interface Verification

Before running Suricata, verify correct interface configuration.

### View Interface Details

```bash
ip a
```

Confirm:
- enp0s3 → 192.168.100.1 (SOC-IN)
- enp0s8 → 192.168.200.1 (SOC-OUT)

---

### Verify Routing Table

```bash
ip r
```

Ensure:
- Routes exist for both subnets
- No misconfigured gateway entries

---

### Verify IP Forwarding (Required for Routing Mode)

```bash
sysctl net.ipv4.ip_forward
```

Expected output:

```
net.ipv4.ip_forward = 1
```

If disabled:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

This ensures traffic flows between SOC-IN and SOC-OUT.

---

## 2. Suricata Configuration Validation

Always validate configuration before starting Suricata.

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

This confirms:

- YAML syntax correctness
- Rule file accessibility
- Rule parsing success
- Output module initialization

Successful output indicates:

```
Configuration provided was successfully loaded.
```

---

## 3. Rule File Verification

### Verify ET Open Rule Count

```bash
wc -l /var/lib/suricata/rules/suricata.rules
```

Confirms large baseline signature set (~48k+ rules).

---

### Verify Custom Rule File

```bash
cat /var/lib/suricata/rules/local.rules
```

Ensures custom rules are present and readable.

---

## 4. Service Status Verification

If running via systemd:

```bash
sudo systemctl status suricata
```

Confirm:
- Service is active (running)
- No recent crash logs

---

## 5. Traffic Traversal Validation (Wire-Level Proof)

Before analyzing alerts, confirm traffic is visible on interfaces.

### Monitor SOC-IN

```bash
sudo tcpdump -i enp0s3
```

### Monitor SOC-OUT

```bash
sudo tcpdump -i enp0s8
```

Generate traffic from Kali (ping or HTTP request) and confirm packets appear on both interfaces.

This validates correct placement of the IDS in the traffic path.

---

## 6. Alert Validation (fast.log)

After triggering an attack scenario:

```bash
sudo tail -n 20 /var/log/suricata/fast.log
```

Confirm:

- Alert SID
- Source IP
- Destination IP
- Signature message

This provides immediate confirmation of detection.

---

## 7. Structured Event Validation (eve.json)

For deeper inspection:

```bash
sudo tail -n 5 /var/log/suricata/eve.json
```

Or structured filtering:

```bash
sudo grep '"alert"' /var/log/suricata/eve.json
```

This confirms:

- Alert metadata
- HTTP URI inspection
- Flow information
- Community ID hash

Structured JSON output demonstrates SIEM-ready telemetry.

---

## 8. HTTP Payload Inspection (For SQLi/XSS Validation)

To visually confirm payload visibility:

```bash
sudo tcpdump -A -i enp0s3 tcp port 80
```

This displays HTTP request content in ASCII.

Used to confirm:

- SQL injection payload presence
- XSS script tag visibility
- Correct packet capture context

---

## 9. Log File Location Summary

| File | Purpose |
|------|----------|
| /var/log/suricata/fast.log | Quick alert proof |
| /var/log/suricata/eve.json | Structured event output |
| /var/log/syslog | Service-level logging |
| /var/lib/suricata/rules/ | Rule storage |

---

## Validation Philosophy

Each attack simulation follows this sequence:

1. Confirm traffic path (tcpdump)
2. Trigger attack payload
3. Verify alert in fast.log
4. Validate structured output in eve.json
5. Confirm rule match (SID reference)

This structured workflow mirrors SOC operational validation procedures.

---

## Summary

These commands ensure:

- Network correctness
- Proper IDS placement
- Successful rule loading
- Reliable alert generation
- Structured evidence output

The validation process emphasizes repeatability and operational accuracy.
