# Configuration Directory

This directory contains the configuration references used in the Suricata IDS Detection Lab.

The purpose of this folder is to document how the IDS was configured, validated, and deployed within the segmented lab environment.

Rather than storing the full production `suricata.yaml` file, this directory includes structured, lab-focused excerpts and documentation to improve clarity and reproducibility.

---

## Directory Purpose

The configuration folder documents:

- Network segmentation design
- Interface assignments and routing model
- Rule loading hierarchy (ET Open + local.rules)
- Logging configuration (fast.log + eve.json)
- Operational validation commands

This ensures reviewers understand how Suricata is deployed and how detections are generated.

---

## Contents

### suricata-config-yaml.md

Describes key sections of the `suricata.yaml` configuration, including:

- HOME_NET / EXTERNAL_NET scoping
- Rule loading paths
- Output modules
- Logging strategy
- Deployment mode (IDS)

This file highlights only lab-relevant configuration elements rather than the full raw YAML.

---

### network-interfaces.md

Documents the network architecture and IP addressing plan, including:

- SOC-IN and SOC-OUT segmentation
- Dual-homed Suricata gateway setup
- Routing enforcement model
- IP forwarding configuration
- Packet-level validation approach

This explains why traffic must traverse Suricata before reaching the target system.

---

### validation-commands.md

Lists structured validation commands used to confirm:

- Interface correctness
- Routing functionality
- Rule loading success
- Alert generation
- Structured log output

This demonstrates operational discipline and repeatable validation workflow.

---

## Live Configuration Location

The active configuration resides on the Suricata VM:

- Main configuration file:
  `/etc/suricata/suricata.yaml`

- Rule directory:
  `/var/lib/suricata/rules/`
  - suricata.rules (ET Open baseline)
  - local.rules (custom lab rules)

- Log output directory:
  `/var/log/suricata/`

The repository intentionally excludes the full raw configuration file to:

- Avoid unnecessary verbosity
- Improve readability
- Focus on detection-relevant configuration elements
- Keep the project concise and reviewer-friendly

---

## Design Philosophy

This configuration documentation emphasizes:

- Clear network scoping
- Deterministic rule loading
- Structured logging
- Reproducibility
- Operational validation discipline

The goal is not just to "run Suricata," but to demonstrate a structured detection engineering workflow.

---

## Summary

The configuration directory provides a transparent view into:

- How the IDS is deployed
- How it inspects traffic
- How rules are applied
- How alerts are validated

It serves as the foundation for the detection and attack simulation workflows documented elsewhere in the repository.
