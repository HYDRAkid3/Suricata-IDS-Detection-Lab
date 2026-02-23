# 02 - Rule Engine Configuration

## Objective

Configure Suricata to load custom detection rules and validate rule parsing.

---

## Custom Rules Location

/etc/suricata/rules/local.rules

---

## Sample Custom Rule (ICMP Test)

```
alert icmp any any -> any any (msg:"ICMP Test Detected"; sid:1000001; rev:1;)
```

---

## Validate Configuration

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Expected:
Configuration provided was successfully loaded.

---

## Run Suricata

```bash
sudo suricata -D -c /etc/suricata/suricata.yaml -i enp0s3
```

---

## Evidence

![Rule Test](../assets/screenshots/rule-engine/01-suricata-test.png)

![Custom Rule](../assets/screenshots/rule-engine/02-local-rules.png)

---

## Findings

- Custom rules loaded successfully
- Suricata validated configuration without errors
- Detection engine operational
