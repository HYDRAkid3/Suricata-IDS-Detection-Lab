# 02 - Rule Engine Configuration

## Objective

Validate Suricata rule loading, confirm ET Open baseline deployment, and demonstrate custom rule development using local.rules.

---

## Rule Sources

Suricata is configured to load rules from:

default-rule-path: /var/lib/suricata/rules

Rule files:
- suricata.rules  (ET Open baseline rules)
- local.rules     (Custom lab rules)

---

## ET Open Rule Verification

### Total Alert Rules Loaded

![ET Rule Count](../assets/screenshots/2%20Rule%20Engine/04_et_rule_count.png)

Approximately 48,000+ alert rules are loaded from the ET Open rule set.

This confirms the IDS is operating with a real-world detection baseline.

---

## Configuration Validation

Command used:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

This verifies:
- Rule parsing success
- No syntax errors
- All rule files properly loaded

---

## Custom Local Rules

![Local Rules Content](../assets/screenshots/2%20Rule%20Engine/05_local_rules_content.png)

Custom rules were developed to support lab-based detection scenarios:

- ICMP path validation
- SYN scan detection
- SQL injection indicators
- Cross-site scripting indicators

These rules supplement ET Open and allow controlled detection validation.

---

## Detection Strategy

The lab uses a layered approach:

1. ET Open baseline signatures (real-world attack detection)
2. Custom local rules (lab-specific validation rules)

This mirrors real SOC environments where baseline signatures are combined with organization-specific detection logic.

---

## Findings

- ET Open successfully installed and active
- 48k+ alert rules available
- Custom rules parsed and loaded
- Rule engine operating correctly
