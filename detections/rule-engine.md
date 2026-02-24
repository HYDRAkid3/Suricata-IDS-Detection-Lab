# 02 - Rule Engine Configuration

## Objective

Validate Suricata’s rule engine configuration, confirm ET Open baseline rule deployment, and demonstrate custom detection rule development using `local.rules`.

---

## Rule Source Configuration

Suricata is configured to load rules from:

default-rule-path:
```
/var/lib/suricata/rules
```

Rule files loaded:
- suricata.rules  → ET Open baseline rule set  
- local.rules     → Custom lab detection rules  

This ensures a layered detection model combining community signatures with lab-specific logic.

---

## ET Open Rule Verification

### Total Alert Rules Loaded

![ET Rule Count](../assets/screenshots/2%20Rule%20Engine/04_et_rule_count.png)

Suricata loaded approximately **48,000+ alert rules** from the ET Open rule set.

This confirms:

- ET Open successfully installed
- Real-world detection signatures active
- IDS operating beyond minimal configuration

---

## Configuration Validation

Rule parsing and configuration validation performed using:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Validation ensures:

- No syntax errors
- All rule files successfully parsed
- Detection engine initialized correctly

---

## Custom Local Rules

![Local Rules Content](../assets/screenshots/2%20Rule%20Engine/05_local_rules_content.png)

Custom rules were developed to support controlled lab validation scenarios:

- ICMP routing path validation
- SYN scan detection (threshold-based)
- SQL Injection indicators
- Cross-Site Scripting (XSS) indicators

These rules complement the ET Open baseline signatures and provide deterministic detection behavior during testing.

---

## Detection Strategy

The rule engine operates using a layered approach:

1. ET Open baseline rules (broad threat detection)
2. Custom local rules (lab-focused validation)

This mirrors real-world SOC deployments where community signatures are supplemented with organization-specific detection logic.

---

## Findings

- ET Open rule set successfully installed
- 48k+ alert rules active
- Custom rules properly parsed and loaded
- Rule engine validated and operational
