# Wireshark Traffic Analysis — CCNA Lab Series
**Equipment:** MacBook Pro 2015 (macOS Monterey 12.7.6) · Cisco 2621 Router · Cisco 1700 Router · Catalyst 3500XL Switch · Linksys E2500 Buffer Router · Insignia USB-A to Gigabit Ethernet Adapter
**Capture Interface:** en4 (USB 10/100/1000 LAN)
**Tool:** Wireshark 4.6.4
**CCNA Domains:** 1.0 Network Fundamentals · 4.0 IP Services · 5.0 Security Fundamentals

---

## Overview

A physical homelab Wireshark series built on real Cisco hardware. Each lab captures live traffic, analyzes it with Wireshark, and documents findings with direct CCNA exam relevance. No simulations — every packet is real.

The lab environment isolates Cisco equipment behind a Linksys E2500 buffer router to prevent experimental configurations from affecting the upstream home network.

---

## Lab Index

| Lab | Title | Key Skill | Status |
|---|---|---|---|
| 01 | [ARP — Address Resolution Protocol](https://github.com/cornerstonian/wireshark-traffic-analysis-ccna/tree/master/lab-01-arp) | ARP request/reply capture, MAC-to-IP mapping, gratuitous ARP | ✅ Complete |
| 02 | [Display Filters — The Force Multiplier](https://github.com/cornerstonian/wireshark-traffic-analysis-ccna/tree/master/lab-02-display-filters) | Protocol isolation, directional filters, logical operators, right-click filter creation | ✅ Complete |
| 03 | [TCP Streams & Cleartext Credential Exposure](https://github.com/cornerstonian/wireshark-traffic-analysis-ccna/tree/master/lab-03-tcp-streams-expert-info) | TCP stream reassembly, Telnet analysis, ROMMON recovery, interface troubleshooting | ✅ Complete |
| 04 | SSH — Encrypted Remote Access | IOS upgrade via TFTP, SSH configuration, encrypted stream comparison | 🔄 In Progress |
| 05 | Switch Integration & VLANs | Catalyst 3500XL, trunk links, VLAN traffic isolation | 📋 Planned |
| 06 | Access Control Lists | ACL implementation, traffic filtering, RST packet analysis | 📋 Planned |

---

## Lab Environment

```
Internet
   ↓
Xfinity Gateway
   ↓
Linksys E2500 (192.168.50.1) — NAT + DHCP + buffer
   ↓
Cisco 2621 Router — NetOps-R1 (192.168.50.10) — Fa0/1
   |
Catalyst 3500XL Switch (planned)
   |
Lab Endpoints
```

| Device | Role | IP |
|---|---|---|
| MacBook Pro 2015 | Capture station / console management | 192.168.50.147 |
| Linksys E2500 | Buffer router — isolates lab from home network | 192.168.50.1 |
| Cisco 2621 (NetOps-R1) | Lab gateway router | 192.168.50.10 |
| Cisco 1700 | Secondary router — SSH lab candidate | TBD |
| Catalyst 3500XL | Core lab switch | Planned |

---

## Capture Files

| File | Lab | Description |
|---|---|---|
| `lab01-arp-cisco2600.pcapng` | 01 | Full ARP exchange with Cisco 2600 |
| `lab02-display-filters-practice.pcapng` | 02 | Multi-protocol capture for filter practice |
| `lab03-telnet-tcp-stream.pcapng` | 03 | Full Telnet session with cleartext credential capture |

Capture files stored locally — not committed to repo due to size.

---

## Skills Demonstrated

- Live packet capture on physical Cisco hardware
- Wireshark display filter construction and saved filter libraries
- TCP stream reassembly and session reconstruction
- Protocol security analysis (Telnet vs SSH)
- ROMMON password recovery and configuration register manipulation
- Interface troubleshooting (Layer 1 / Layer 2 status diagnosis)
- Remote access configuration (VTY lines, transport input)

---

*Lavoisier Cornerstone — [lavoisier.dev](https://lavoisier.dev) | [github.com/cornerstonian](https://github.com/cornerstonian)*
*Part of the ccna-wireshark-labs project series*
