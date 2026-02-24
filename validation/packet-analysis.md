# Packet Analysis & Traffic Path Validation

This document validates that Suricata is correctly positioned inline between attacker and victim networks and is capable of observing, forwarding, and inspecting live traffic.

All validation screenshots are located in:

evidence/screenshots/

---

# 1. Inline Routing Verification

Before analyzing attacks, we first confirm that traffic must traverse Suricata.

Kali default gateway configuration:

![Kali Default Gateway via Suricata](../evidence/screenshots/03_Network_Path_Validation/01_kali_default_gateway_suricata.png)

Key observation:
- Kali IP: 192.168.100.10
- Default Gateway: 192.168.100.1 (Suricata ingress interface)

This ensures traffic to 192.168.200.0/24 cannot bypass the IDS.

---

# 2. Interface Visibility Validation

Traffic capture on Suricata ingress interface (SOC-IN):

![Ingress Packet Capture](../evidence/screenshots/03_Network_Path_Validation/02_suricata_enp0s3_ingress_capture.png)

This confirms:
- Packets from 192.168.100.10 enter enp0s3
- Suricata observes inbound attacker traffic

---

# 3. HTTP Payload Transit Validation

Captured HTTP payload during exploitation attempt:

![HTTP Payload Transit](../evidence/screenshots/03_Network_Path_Validation/03_suricata_http_payload_transit.png)

Observations:
- GET request visible in packet payload
- Application-layer inspection enabled
- Clear visibility of malicious parameters

This confirms Suricata is inspecting Layer 7 traffic.

---

# 4. Reconnaissance Traffic Validation

## Host Discovery (ICMP)

![Host Discovery Ping](../evidence/screenshots/04_Recon_Scan/00_host_discovery_ping.png)

Confirms:
- Cross-subnet ICMP routing operational
- Suricata forwarding traffic successfully

---

## SYN Scan Activity

![Nmap SYN Scan](../evidence/screenshots/04_Recon_Scan/01_nmap_syn_scan.png)

This produces:
- TCP SYN packets
- Incomplete handshakes
- Reconnaissance signatures

---

## Kali IP and Route Confirmation

![Kali Route Validation](../evidence/screenshots/04_Recon_Scan/02_recon_kali_ip_route_ping_target.png)

Ensures:
- Proper subnet segmentation
- Forced inspection boundary

---

# 5. Web Server Exposure Validation

## HTTP Header Enumeration

![HTTP Header & Banner](../evidence/screenshots/04_Recon_Scan/03_recon_http_headers_and_banner.png)

Reveals:
- Apache version
- Ubuntu OS indicator
- Web service availability

---

## TCP Connect Scan

![Nmap TCP Connect Scan](../evidence/screenshots/04_Recon_Scan/04_recon_nmap_tcp_connect_scan.png)

Confirms:
- Port 80 open
- Service reachable
- Recon stage complete

---

## Application Fingerprinting

![Application Fingerprint](../evidence/screenshots/04_Recon_Scan/05_recon_app_fingerprint_strings.png)

Shows:
- Apache2 default page
- Ubuntu server details
- Technology stack confirmation

---

## Port Status Verification

![HTTP/HTTPS Port Status](../evidence/screenshots/04_Recon_Scan/06_recon_port_status_http_https.png)

Validates:
- HTTP open
- HTTPS closed/refused
- Attack surface identified

---

# 6. SQL Injection Packet-Level Analysis

## DVWA Security Level

![DVWA Security Low](../evidence/screenshots/05_SQLi_Case_Study/00_dvwa_security_level_low.png)

Ensures:
- Vulnerability exposure enabled

---

## Web Server Reachability

![DVWA Reachable](../evidence/screenshots/05_SQLi_Case_Study/01_dvwa_webserver_reachable.png)

Confirms:
- Web application accessible
- Target service operational

---

## Boolean Injection Success

![SQLi Boolean Bypass](../evidence/screenshots/05_SQLi_Case_Study/02_dvwa_sqli_boolean_bypass_success.png)

Demonstrates:
- Successful exploitation
- Malicious payload transit across Suricata

---

# 7. XSS Packet-Level Analysis

## XSS Payload in Transit

![HTTP XSS Payload Packet Capture](../evidence/screenshots/06_XSS_Case_Study/01_http_xss_payload_packet_capture.png)

Confirms:
- Script payload visible in HTTP request
- Layer 7 inspection active

---

# 8. Summary

Packet-level validation confirms:

✔ Suricata correctly positioned inline  
✔ Dual-interface routing operational  
✔ Layer 3 forwarding functional  
✔ Layer 4 reconnaissance visible  
✔ Layer 7 payload inspection confirmed  
✔ Attack traffic fully observable  

This proves the IDS is correctly deployed in an inspection boundary configuration.
