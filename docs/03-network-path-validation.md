# 03 - Network Path Validation

## Objective

Confirm all traffic between attacker and victim passes through Suricata.

---

## ICMP Test

From Kali:

```bash
ping 192.168.200.10
```

On Suricata:

```bash
tcpdump -i enp0s3 icmp
tcpdump -i enp0s8 icmp
```

---

## Expected Behavior

- ICMP visible on both interfaces
- Alert generated if ICMP rule exists

---

## Evidence

![ICMP Traffic](../assets/screenshots/network-path/01-icmp.png)

![fast.log ICMP Alert](../assets/screenshots/network-path/02-fastlog-icmp.png)

---

## Findings

- Traffic confirmed flowing through IDS
- Routing enforcement validated
- Inspection path verified
