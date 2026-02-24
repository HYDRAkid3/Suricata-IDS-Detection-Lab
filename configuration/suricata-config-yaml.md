# Suricata Configuration (suricata.yaml) – Lab Tuning & Key Settings

This document explains the key `suricata.yaml` settings used in this lab, why they matter, and how they support inline inspection across the segmented SOC-IN and SOC-OUT networks.

Lab context:
- Attacker (Kali): 192.168.100.10
- Suricata (Gateway/IDS): 192.168.100.1 (enp0s3), 192.168.200.1 (enp0s8)
- Target (DVWA): 192.168.200.10
- SOC-IN: 192.168.100.0/24
- SOC-OUT: 192.168.200.0/24

The goal of this configuration is to ensure:
- Both interfaces are monitored
- HTTP payloads are parsed correctly (app-layer detection)
- ET Open + custom rules are loaded successfully
- Alerts are written to both `fast.log` and `eve.json`
- Configuration validation is repeatable and error-free

## 1) Configuration Validation (Sanity Check)

Before running Suricata in live mode, the config was validated using test mode to catch YAML syntax errors, missing rule files, and misconfigured outputs.

Command:
sudo suricata -T -c /etc/suricata/suricata.yaml

![Suricata Configuration Validation](../evidence/screenshots/01_Infrastructure_Setup/05_configuration_validation_success.png)

Expected outcome:
- “Configuration provided was successfully loaded”
- Suricata exits cleanly (test mode success)

If this fails, do not proceed to runtime capture until the errors are fixed.

## 2) Interfaces & Capture Mode (Multi-Interface Monitoring)

This lab requires Suricata to monitor traffic crossing the boundary between SOC-IN and SOC-OUT. That means Suricata must see traffic on both interfaces:

- enp0s3 (SOC-IN side, 192.168.100.1)
- enp0s8 (SOC-OUT side, 192.168.200.1)

Interface state proof (dual-homed gateway):

![Suricata Interfaces Up](../evidence/screenshots/01_Infrastructure_Setup/03_suricata_interfaces_up.png)

In `suricata.yaml`, packet acquisition is configured so Suricata can capture traffic from the relevant interfaces. Depending on your environment, capture can be implemented using `af-packet`, `pcap`, or another supported capture method.

What matters for this project:
- Suricata must capture traffic on the interface(s) where transit packets pass
- Ingress traffic from the attacker subnet must be visible
- HTTP requests reaching the DVWA server must be visible and decoded at app-layer

Runtime proof that Suricata initialized capture and engine successfully:
![Suricata Engine Initialized](../evidence/screenshots/01_Infrastructure_Setup/06_suricata_engine_initialized.png)

Thread model proof (capture + detect threads created):
![Suricata Runtime Threads](../evidence/screenshots/01_Infrastructure_Setup/07_suricata_runtime_threads.png)

Key validation point:
If Suricata starts but does not show capture initialization or thread creation, detection will not occur.

## 3) IP Forwarding Requirement (Gateway Function)

Although Suricata runs in IDS mode (detect-only), the VM is also acting as a router between SOC-IN and SOC-OUT. This requires IPv4 forwarding enabled at the OS level.

Proof of forwarding enabled:

![IP Forwarding Enabled](../evidence/screenshots/01_Infrastructure_Setup/04_ip_forwarding_enabled.png)

Why it matters:
- If forwarding is disabled, traffic will not traverse between subnets
- Recon and exploitation traffic will fail or bypass expected transit behavior
- Detection validation becomes unreliable

## 4) Network Variables (HOME_NET / EXTERNAL_NET)

Suricata rules rely heavily on correct variable scoping, especially for “internal” vs “external” definitions.

Recommended lab mapping:
- HOME_NET should represent protected/internal networks (SOC-IN and SOC-OUT)
- EXTERNAL_NET should represent everything else

In a two-subnet lab, the safest approach is to treat both internal ranges as HOME_NET, so web attacks and scans across the gateway are evaluated as internal boundary traffic.

Operational impact:
- Many ET rules use `$HOME_NET -> $EXTERNAL_NET` or the reverse
- Incorrect HOME_NET causes missed detections or noisy false positives

## 5) Rule Loading (ET Open + Custom Local Rules)

This lab uses two rule sources:
- ET Open rule set (baseline coverage)
- Custom local rules (lab-specific signatures for ICMP, SYN scan, SQLi, XSS)

The `suricata.yaml` configuration must include the correct `rule-files` list and reference the correct local rules path.

Proof that local rules are included in configuration:
![Local Rules Loaded in Config](../evidence/screenshots/02_Rule_Engine_Validation/01_local_rules_loaded_in_config.png)

Proof that Suricata successfully processed and loaded rule sets:
![Rules Loaded Successfully](../evidence/screenshots/02_Rule_Engine_Validation/02_suricata_rules_loaded_successfully.png)

Baseline rule statistics proof (ET Open present and active):
![ET Open Rule Base Statistics](../evidence/screenshots/02_Rule_Engine_Validation/04_rule_base_statistics_et_open.png)

What “success” looks like:
- Rule files processed: count > 0
- Rules successfully loaded: high count
- Failed rules: 0 (or explicitly understood and documented)
- No “cannot open rule file” or “no such file” errors

## 6) Output Configuration (fast.log and eve.json)

This project validates detections using:
- fast.log (quick alert proof, easy screenshots)
- eve.json (structured output with HTTP metadata, SIDs, URIs, signatures)

Your `suricata.yaml` must ensure:
- `fast.log` output enabled
- `eve-log` output enabled
- HTTP logging enabled inside eve configuration (so URI, method, host, user-agent appear)

Why both outputs matter:
- fast.log proves “alert fired”
- eve.json proves “alert context” (who, what, where, which URI, which signature_id)

This is essential for SOC-style alert analysis and correlation.

## 7) App-Layer Protocol Detection (HTTP Visibility)

The SQLi and XSS detections in this project depend on Suricata correctly parsing HTTP requests (Layer 7). That requires:
- App-layer enabled (default in modern Suricata)
- HTTP protocol detection functional
- Sufficient capture depth / packet reassembly settings (default is usually adequate for DVWA-style payloads)

Validation outcome:
If HTTP is parsed correctly, eve.json alerts include fields like:
- http.method
- http.uri
- http.hostname
- http.user_agent

This is what enables reliable detection and explainable alerts.

## 8) Operational Checks (What to verify every run)

Before running attacks, validate these checkpoints:

1) Config test mode:
sudo suricata -T -c /etc/suricata/suricata.yaml
Expected: success (no errors)

2) Interfaces up and IPs correct:
enp0s3 = 192.168.100.1
enp0s8 = 192.168.200.1

3) Forwarding enabled:
cat /proc/sys/net/ipv4/ip_forward
Expected: 1

4) Rules loaded:
Suricata startup shows rules processed successfully and no rule file errors

5) Outputs enabled:
- /var/log/suricata/fast.log exists and updates on alerts
- /var/log/suricata/eve.json exists and contains event_type alerts

## 9) Summary

This `suricata.yaml` configuration supports the lab objectives by ensuring:
- Dual-interface traffic visibility (SOC-IN and SOC-OUT)
- Stable, validated startup using test mode
- Correct routing enforcement via IP forwarding
- ET Open baseline detection + custom rule coverage
- Alert output in both fast.log and eve.json
- Application-layer (HTTP) parsing for SQLi and XSS detection

With these settings validated, reconnaissance scans and web exploitation payloads reliably generate alerts that can be documented and analyzed in later sections of the repository.
