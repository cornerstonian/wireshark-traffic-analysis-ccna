# Wireshark Traffic Analysis — CCNA Lab Series
**Equipment:** MacBook Pro 2015 (macOS Monterey 12.7.6) · Cisco 2621 Router · Cisco 1700 Router · Catalyst 3500XL Switch · Linksys E2500 Buffer Router · Insignia USB-A to Gigabit Ethernet Adapter
**Capture Interface:** en4 (USB 10/100/1000 LAN)
**Tool:** Wireshark 4.6.4
**CCNA Domains:** 1.0 Network Fundamentals · 2.0 Network Access · 4.0 IP Services · 5.0 Security Fundamentals

---

## Overview

A physical homelab Wireshark series built on real Cisco hardware. Each lab captures live traffic, analyzes it with Wireshark, and documents findings with direct CCNA exam relevance. No simulations — every packet is real.

The lab environment isolates Cisco equipment behind a Linksys E2500 buffer router to prevent experimental configurations from affecting the upstream home network. Lab 05 introduced the Catalyst 3500XL as the core lab switch, adding VLAN segmentation, ISL trunking, and inter-VLAN routing to the topology. All subsequent labs build on this infrastructure.

> **Numbering convention:** Lab numbers reflect execution order on physical gear — not book curriculum order. The original book curriculum has been reconciled to this equipment, this topology, and this IOS version throughout.

---

## Lab Index — Completed

| Lab | Title | Key Skill | Status |
|---|---|---|---|
| 01 | [ARP — Address Resolution Protocol](https://github.com/cornerstonian/wireshark-traffic-analysis-ccna/tree/master/lab-01-arp) | ARP request/reply capture, MAC-to-IP mapping, gratuitous ARP | ✅ Complete |
| 02 | [Display Filters — The Force Multiplier](https://github.com/cornerstonian/wireshark-traffic-analysis-ccna/tree/master/lab-02-display-filters) | Protocol isolation, directional filters, logical operators, right-click filter creation | ✅ Complete |
| 03 | [TCP Streams & Cleartext Credential Exposure](https://github.com/cornerstonian/wireshark-traffic-analysis-ccna/tree/master/lab-03-tcp-streams-expert-info) | TCP stream reassembly, Telnet analysis, ROMMON recovery, interface troubleshooting | ✅ Complete |
| 04 | [SSH — Encrypted Remote Access](https://github.com/cornerstonian/wireshark-traffic-analysis-ccna/tree/master/lab-04-ssh-encrypted-remote-access) | SSH configuration, RSA key generation, encrypted stream analysis, Telnet vs SSH comparison | ✅ Complete |
| 05 | [Switch Integration & VLANs](https://github.com/cornerstonian/wireshark-traffic-analysis-ccna/tree/master/lab-05-switch-integration-vlans) | Catalyst 3500XL integration, VLAN segmentation, ISL trunking, router-on-a-stick, STP capture | ✅ Complete |

---

## Lab Index — Planned

| Lab | Title | Key Skill | Priority |
|---|---|---|---|
| 06 | Access Control Lists | Extended ACL on 2621, inter-VLAN traffic filtering, RST capture, ICMP Admin Prohibited (type 3 code 13) | Critical |
| 07 | DNS — Query Analysis & Problems | DNS query/response lifecycle, NXDOMAIN, TTL analysis, UDP vs TCP fallback — captured through VLAN topology | Critical |
| 08 | ICMP — Reachability & Diagnostics | Echo request/reply, TTL expiry via traceroute across VLANs, ACL-generated ICMP unreachable | Critical |
| 09 | TCP — Deep Dive & Stream Analysis | 3-way handshake, sequence numbers, RST from ACL deny, retransmission analysis, FIN teardown | Critical |
| 10 | UDP — Behavior & Problems | DNS over UDP, DHCP DORA process, UDP silent failure vs TCP RST comparison | High |
| 11 | Statistics & IO Graphs | Protocol hierarchy across VLAN topology, IO graph broadcast detection, conversation analysis | High |
| 12 | Capstone — Full Troubleshooting Scenario | Diagnose 4 injected faults from Wireshark captures only — no CLI allowed during diagnosis | Capstone |

---

## Lab Environment

### Current Topology (Lab 05+)

```
Internet
   ↓
Xfinity Gateway
   ↓
Linksys E2500 (192.168.50.1) — NAT + DHCP + buffer
   ↓
Catalyst 3500XL — NetOps-SW1 (192.168.50.30) — core lab switch
   │
   ├── Fa0/1  [VLAN 1]    → Linksys E2500 (uplink)
   ├── Fa0/2  [VLAN 10]   → MacBook Pro (en4) — 192.168.10.50
   ├── Fa0/3  [TRUNK/ISL] → Cisco 2621 — NetOps-R1 (Fa0/0)
   └── Fa0/4  [VLAN 20]   → Cisco 1700 — NetOps-1700 (Fa0)
```

### VLAN Design

| VLAN | Name | Subnet | Purpose |
|---|---|---|---|
| 1 | default | 192.168.50.0/24 | Switch management + E2500 uplink |
| 10 | MANAGEMENT | 192.168.10.0/24 | MacBook capture station |
| 20 | ROUTERS | 192.168.20.0/24 | Cisco 2621 + Cisco 1700 |

### Device Table

| Device | Role | IP | VLAN |
|---|---|---|---|
| MacBook Pro 2015 | Capture station / console management | 192.168.10.50 (en4) | 10 |
| Linksys E2500 (NetOpsZero) | Buffer router — NAT + DHCP | 192.168.50.1 | 1 |
| Catalyst 3500XL (NetOps-SW1) | Core lab switch | 192.168.50.30 | 1 (mgmt) |
| Cisco 2621 (NetOps-R1) | Inter-VLAN router — Fa0/0.10 / Fa0/0.20 | 192.168.10.1 / 192.168.20.1 | Trunk |
| Cisco 1700 (NetOps-1700) | SSH lab device — advsecurityk9 IOS 12.4(25d) | 192.168.20.20 | 20 |

### Pre-Lab 05 Topology (Labs 01–04, archived)

```
Internet
   ↓
Xfinity Gateway
   ↓
Linksys E2500 (192.168.50.1) — NAT + DHCP + buffer
   ↓
   ├── Cisco 2621 — NetOps-R1 (192.168.50.10) — Fa0/0
   │   Labs 01–03 — ARP / display filters / Telnet cleartext
   │
   └── Cisco 1700 — NetOps-1700 (192.168.50.20) — Fa0
       Lab 04 — SSH / encrypted remote access
```

---

## Capture Files

| File | Lab | Description |
|---|---|---|
| `lab01-arp-cisco2600.pcapng` | 01 | Full ARP exchange with Cisco 2621 |
| `lab02-display-filters-practice.pcapng` | 02 | Multi-protocol capture for filter practice |
| `lab03-telnet-tcp-stream.pcapng` | 03 | Full Telnet session with cleartext credential capture |
| `lab04-ssh-encrypted-stream.pcapng` | 04 | SSH session — encrypted stream, no readable credentials |
| `lab05-vlan-isolation.pcapng` | 05 | ARP broadcasts with zero replies — VLAN isolation proof |
| `lab05-intervlan-routing.pcapng` | 05 | ICMP across VLANs — TTL=254 confirms routed hop through 2621 |
| `lab05-stp-bpdus.pcapng` | 05 | STP BPDUs from NetOps-SW1 — 2 second hello interval, root bridge confirmed |

Capture files stored locally — not committed to repo due to size.

---

## Planned Lab Scope Notes

The following notes reconcile the original 10-lab book curriculum to current hardware, topology, and execution state.

**Lab 06 — Access Control Lists**
Extended ACLs applied on the 2621 between VLANs — filtering traffic from VLAN 10 to VLAN 20 and capturing the results. ICMP Admin Prohibited (type 3, code 13) is the primary Wireshark deliverable. RST packets from denied TCP sessions are the secondary. The existing inter-VLAN routing infrastructure from Lab 05 makes this lab richer than the flat-network version in the original curriculum.

**Lab 07 — DNS Query Analysis & Problems**
DNS captured on en4 through the lab switch, giving a realistic view of DNS traversing a switched VLAN infrastructure. NXDOMAIN, TTL caching, and TCP fallback are the core capture targets. No new hardware required. Original curriculum used Wi-Fi/en0 — en4 through the VLAN topology is the correct path for this lab series.

**Lab 08 — ICMP Reachability & Diagnostics**
Traceroute across VLANs reveals the 2621 as a hop, producing Time Exceeded messages. TTL decrement analysis from Lab 05 carries forward directly. The ACL from Lab 06 can be reactivated to capture Admin Prohibited in context. TTL values differ from the original curriculum because the VLAN topology adds a routing hop.

**Lab 09 — TCP Deep Dive & Stream Analysis**
Sequence number tracking across the 3-way handshake, RST from ACL deny, retransmission simulation, graceful FIN teardown. The Telnet (2621) and SSH (1700) infrastructure from Labs 03 and 04 is reused. No new configuration required. Original curriculum covered this as Lab 07 — execution order places it here after ACL, DNS, and ICMP.

**Lab 10 — UDP Behavior & Problems**
DNS over UDP, DHCP DORA process visible on the 192.168.50.x segment via the E2500, UDP silent failure vs TCP RST comparison. No new hardware required. Original curriculum covered this as Lab 08.

**Lab 11 — Statistics & IO Graphs**
Uses captures from all previous labs. Protocol hierarchy across the full VLAN topology. IO graph for broadcast storm identification using the STP capture from Lab 05 as baseline. Conversation analysis showing inter-VLAN traffic pairs. No new captures required. Original curriculum covered this as Lab 09.

**Lab 12 — Capstone**
Four injected faults diagnosed from Wireshark alone — no show commands during diagnosis phase. Fault scenarios updated from the original curriculum for the VLAN topology: ARP failure (clear MAC table on 3500XL), DNS failure (misconfigure resolver on MacBook), ACL blocking inter-VLAN routing (deny on 2621 between VLANs), DHCP exhaustion (restrict E2500 pool). Full topology with all devices active. Original curriculum covered this as Lab 10.

---

## Skills Demonstrated

**Completed**
- Live packet capture on physical Cisco hardware
- Wireshark display filter construction and saved filter libraries
- TCP stream reassembly and session reconstruction
- Protocol security analysis — Telnet cleartext vs SSH encrypted
- SSH configuration — RSA key generation, VTY hardening, version 2 enforcement
- macOS OpenSSH legacy algorithm compatibility (`~/.ssh/config` permanent fix)
- ROMMON password recovery and configuration register manipulation
- Interface troubleshooting (Layer 1 / Layer 2 status diagnosis)
- Remote access configuration (VTY lines, transport input)
- VLAN creation on Cisco Catalyst 3500XL (IOS 12.0 `vlan database` mode)
- ISL trunk link configuration between switch and router
- Router-on-a-stick inter-VLAN routing via subinterfaces
- VLAN isolation proof via Wireshark ARP capture
- Inter-VLAN routing proof via TTL decrement analysis
- STP BPDU capture and root bridge identification
- macOS network service order configuration for dual-interface lab operation

**Planned**
- Extended ACL configuration and Wireshark verification (ICMP Admin Prohibited, RST)
- DNS packet analysis — query/response lifecycle, NXDOMAIN, TTL, TCP fallback
- ICMP type/code analysis — echo, unreachable, time exceeded, traceroute hop mapping
- TCP deep dive — sequence numbers, handshake, retransmissions, RST, FIN teardown
- UDP behavior — connectionless capture, silent failure, DNS/DHCP/TFTP analysis
- Wireshark statistics — protocol hierarchy, IO graphs, conversation analysis
- Full troubleshooting capstone — fault injection and packet-level diagnosis, no CLI

---

*Lavoisier Cornerstone — [lavoisier.dev](https://lavoisier.dev) | [github.com/cornerstonian](https://github.com/cornerstonian)*
*Part of the wireshark-traffic-analysis-ccna project series*
