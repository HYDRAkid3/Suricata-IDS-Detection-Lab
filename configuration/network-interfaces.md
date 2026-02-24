# Lab Interfaces & IP Plan

## VirtualBox Networks
- SOC-IN  : 192.168.100.0/24
- SOC-OUT : 192.168.200.0/24

## Machines

### Kali (Attacker)
- Network: SOC-IN
- IP: 192.168.100.10/24
- Gateway: 192.168.100.1

### Suricata IDS VM (Routing Inspection Gateway)
- enp0s3 (SOC-IN)  : 192.168.100.1/24
- enp0s8 (SOC-OUT) : 192.168.200.1/24
- IP Forwarding: enabled (net.ipv4.ip_forward = 1)

Traffic path enforced:
Kali → Suricata → DVWA

### Ubuntu DVWA (Victim Web Server)
- Network: SOC-OUT
- IP: 192.168.200.10/24
- Gateway: 192.168.200.1
- Service: Apache + DVWA (HTTP)

## Verification Commands (Reference)

On Suricata:
- `ip a`
- `ip r`
- `sysctl net.ipv4.ip_forward`
- `tcpdump -i enp0s3`
- `tcpdump -i enp0s8`
