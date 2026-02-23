# pcaps/

This folder contains small packet captures used as evidence for the Suricata IDS Detection Lab.

## Why PCAPs are included
- Validate that traffic traverses the Suricata routing gateway (SOC-IN ↔ SOC-OUT)
- Provide wire-level evidence for HTTP payloads used in SQLi/XSS demonstrations
- Support reproducibility for reviewers

## Files
- `01-icmp-path-validation.pcap`  
  ICMP traffic showing routed traversal through the IDS.

- `02-http-sqli-payload.pcap`  
  HTTP request capture containing SQLi payload indicators.

- `03-http-xss-payload.pcap`  
  HTTP request capture containing XSS payload indicators.

## Notes
- PCAPs are intentionally short (few seconds) to keep the repo lightweight.
- Captures were taken on the Suricata VM using `tcpdump`.
