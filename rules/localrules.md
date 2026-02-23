# Suricata IDS Detection Lab - Custom Rules

Author: Harshit Krishna  
Purpose: Custom detection rules for lab validation

---

## Overview

This document contains the custom Suricata rules created to validate detection scenarios within the controlled lab environment.

These rules supplement the ET Open baseline rule set and are scoped specifically to:

- ICMP traffic validation
- SYN scan detection
- SQL Injection attempts
- Cross-Site Scripting attempts

---

## ET Baseline Verification

![ET Rule Count](../assets/screenshots/2%20Rule%20Engine/04_et_rule_count.png)

The IDS is operating with approximately 48,000+ ET Open alert rules, providing real-world detection coverage.

---

## Custom Rule File Content

![Local Rules Screenshot](../assets/screenshots/2%20Rule%20Engine/05_local_rules_content.png)

---

## Custom Rules (Reference Copy)

```rules
# ICMP Path Validation
alert icmp any any -> any any (
    msg:"LAB ICMP Traffic Observed (Path Validation)";
    itype:8;
    classtype:network-event;
    sid:1000001;
    rev:1;
)

# SYN Scan Detection
alert tcp any any -> 192.168.200.0/24 any (
    msg:"LAB Possible SYN Scan Detected";
    flags:S;
    flow:stateless;
    detection_filter:track by_src, count 20, seconds 10;
    classtype:attempted-recon;
    sid:1000010;
    rev:1;
)

# SQL Injection - OR 1=1
alert http any any -> 192.168.200.10 80 (
    msg:"LAB SQL Injection Attempt - OR 1=1 Pattern";
    flow:to_server,established;
    http.uri;
    content:"or 1=1"; nocase;
    classtype:web-application-attack;
    sid:1000020;
    rev:1;
)

# SQL Injection - UNION
alert http any any -> 192.168.200.10 80 (
    msg:"LAB SQL Injection Attempt - UNION SELECT";
    flow:to_server,established;
    http.uri;
    content:"union"; nocase;
    classtype:web-application-attack;
    sid:1000021;
    rev:1;
)

# XSS Detection - Script Tag
alert http any any -> 192.168.200.10 80 (
    msg:"LAB XSS Attempt - Script Tag Detected";
    flow:to_server,established;
    http.uri;
    content:"<script"; nocase;
    classtype:web-application-attack;
    sid:1000030;
    rev:1;
)

# XSS Detection - Encoded Script
alert http any any -> 192.168.200.10 80 (
    msg:"LAB Encoded XSS Attempt Detected";
    flow:to_server,established;
    http.uri;
    content:"%3Cscript%3E"; nocase;
    classtype:web-application-attack;
    sid:1000031;
    rev:1;
)
```

---

## Design Notes

- Custom SIDs start at 1000000+ to avoid conflicts with ET rules.
- Rules are scoped to the lab subnet (192.168.200.0/24).
- Thresholding is used for scan detection to reduce noise.
- Rules are intentionally simple for deterministic validation.

---

## Summary

The lab rule engine combines:

- ET Open baseline detection (community signatures)
- Controlled custom rule validation
- Structured detection workflow

This approach reflects practical SOC detection architecture.
