# 02 - Rule Engine Configuration

## Objective

Load and validate custom detection rules in Suricata.

---

## Local Rules Configuration

![Local Rules](../assets/screenshots/2%20Rule%20Engine/01_local_rules_configuration.png)

Custom rules defined in:

/etc/suricata/rules/local.rules

---

## Rules Loaded Successfully

![Rules Loaded](../assets/screenshots/2%20Rule%20Engine/02_rules_loaded_successfully.png)

Suricata confirmed successful rule loading.

---

## Alert Proof (fast.log)

![fast.log Alert](../assets/screenshots/2%20Rule%20Engine/03_fast_log_alert_proof.png)

Alert successfully triggered and logged.

---

## Findings

- Custom rules parsed correctly
- Detection engine functioning
- Alerts written to fast.log
