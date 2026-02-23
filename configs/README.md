# configs/

This folder contains lab configuration references for the Suricata IDS Detection Lab.

## What is included

- `suricata-lab-snippets.yaml`  
  A sanitized excerpt of the Suricata configuration showing only lab-relevant settings (networks, rule paths, and outputs).

- `interfaces.md`  
  Network interfaces, IPs, and routing assumptions for the dual-network inspection gateway.

## Notes

The live Suricata configuration is located on the IDS VM at:

- `/etc/suricata/suricata.yaml` (full configuration)
- Rules loaded from `/var/lib/suricata/rules/` (ET Open + local.rules)

This repo intentionally stores only the minimal config needed for reproducibility and reviewer clarity.
