# Reconnaissance Phase – Network & Service Enumeration

This section documents reconnaissance activity performed from the attacker machine (Kali Linux) against the DVWA target host.

Attacker: 192.168.100.10  
Target: 192.168.200.10  
Inline Inspection Node: Suricata (Gateway between SOC-IN and SOC-OUT)

The objective of this phase was to:
- Identify live hosts
- Discover open ports
- Fingerprint exposed services
- Validate Suricata detection of reconnaissance activity

## 1) Network Context & Routing Validation

Before active scanning, Kali’s network configuration and routing path were validated to ensure traffic flows through the Suricata inline inspection node.

![Kali IP & Route Validation](../evidence/screenshots/04_Recon_Scan/02_recon_kali_ip_route_ping_target.png)

Key observations:
- Kali IP: 192.168.100.10
- Default gateway: 192.168.100.1 (Suricata ingress interface)
- Traffic destined for 192.168.200.0/24 must traverse the inspection node

This confirms that attacker traffic cannot directly reach the target without passing through Suricata.

## 2) Host Discovery (ICMP)

A basic ICMP probe was used to confirm that the target system is reachable.

Command executed:
ping -c 4 192.168.200.10

![Host Discovery Ping](../evidence/screenshots/04_Recon_Scan/00_host_discovery_ping.png)

Result:
The target responded successfully with 0% packet loss, confirming Layer 3 connectivity between network segments via the Suricata gateway.

## 3) Stealth Port Discovery (SYN Scan)

A TCP SYN scan was performed to identify reachable services while minimizing full TCP handshake completion.

Command executed:
nmap -sS -p 80 192.168.200.10

![Nmap SYN Scan](../evidence/screenshots/04_Recon_Scan/01_nmap_syn_scan.png)

Observation:
Port 80/tcp was identified as open (SYN-ACK received).

This confirms:
- Web service is exposed
- Target is actively listening
- TCP communication is functioning

## 4) Full TCP Connect Validation

A TCP connect scan was used to validate full session establishment capability.

Command executed:
sudo nmap -n -Pn -sT -p 80 --reason 192.168.200.10

![TCP Connect Scan](../evidence/screenshots/04_Recon_Scan/04_recon_nmap_tcp_connect_scan.png)

Result:
Port 80 confirmed open with reason “syn-ack”, verifying service availability.

## 5) HTTP Banner Grabbing

Banner grabbing was performed to identify the web server software and confirm application-layer access.

Command executed:
curl -I http://192.168.200.10/

![HTTP Headers & Banner](../evidence/screenshots/04_Recon_Scan/03_recon_http_headers_and_banner.png)

Important headers identified:
- Server: Apache/2.4.58 (Ubuntu)
- HTTP/1.1 200 OK
- Content-Type: text/html

This reveals:
- Apache web server running
- Ubuntu-based environment
- HTTP service accessible without TLS

## 6) Application Fingerprinting

Further inspection of the HTTP response body was conducted.

Command executed:
curl http://192.168.200.10/ | head

![Application Fingerprint](../evidence/screenshots/04_Recon_Scan/05_recon_app_fingerprint_strings.png)

The Apache2 Ubuntu default page was observed, indicating:
- Default server configuration
- No obfuscation or hardening
- Likely development or lab deployment

## 7) Port Status Validation (HTTP vs HTTPS)

Manual validation using Netcat:

Command executed:
nc -vz -w 2 192.168.200.10 80  
nc -vz -w 2 192.168.200.10 443

![Port Status HTTP HTTPS](../evidence/screenshots/04_Recon_Scan/06_recon_port_status_http_https.png)

Results:
- Port 80 (HTTP): Open
- Port 443 (HTTPS): Connection refused

Security implication:
All web traffic is transmitted in cleartext, enabling full application-layer inspection by Suricata.

## 8) IDS Detection of Reconnaissance Activity

During scanning activity, Suricata generated detection alerts including:

- SYN SCAN DETECTED
- ET SCAN Possible Nmap User-Agent Observed

These alerts confirm:
- ET Open rule set operational
- TCP scanning behavior identified
- Inline detection functioning correctly

## MITRE ATT&CK Mapping

Host Discovery – T1018 (Remote System Discovery)  
Port Scanning – T1046 (Network Service Scanning)  
Service Enumeration – T1082 (System Information Discovery)

## Reconnaissance Phase Summary

The reconnaissance phase successfully identified:
- A live target host at 192.168.200.10
- Open service on port 80 (HTTP)
- Apache/Ubuntu web stack
- No HTTPS enabled
- Detectable scanning behavior

This validates:
- Correct Suricata inline placement
- End-to-end traffic visibility
- Network and application layer inspection capability
- Functional IDS detection for reconnaissance techniques
