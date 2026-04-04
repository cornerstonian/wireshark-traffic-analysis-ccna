# Display Filters — The Force Multiplier

**Lab:** 02 — Display Filters  
**Equipment:** MacBook Pro 2015 (macOS Monterey 12.7.6) · Cisco 2600 Router · Linksys E2500 Buffer Router · Insignia USB-A to Gigabit Ethernet Adapter  
**Tool:** Wireshark 4.6.4  
**Capture Interface:** en4 (USB 10/100/1000 LAN)  
**Capture File:** `lab02-display-filters-practice.pcapng` (stored locally)  
**CCNA Domain:** 1.0 Network Fundamentals — Protocol identification, traffic isolation, troubleshooting methodology  

---

## Overview

Display filters are the most important Wireshark skill. A capture with 50,000 packets is useless without the ability to isolate exactly the traffic that matters. This lab builds filter muscle memory across protocol isolation, directional filtering, logical operators, right-click filter creation, and saved filter libraries.

Display filters apply to already-captured packets — they hide non-matching packets without deleting them. The full capture remains intact underneath.

**Capture filters** (set before capture) vs **display filters** (applied after capture):
- Capture filters use BPF syntax: `host 192.168.50.10`, `port 80`
- Display filters use Wireshark syntax: `ip.addr == 192.168.50.10`, `tcp.port == 80`
- This lab focuses entirely on display filters

---

## Lab Environment

| Component | Device | IP Address |
|---|---|---|
| Capture Station | MacBook Pro 2015 — macOS Monterey 12.7.6 | 192.168.50.147 |
| Capture Interface | Insignia USB-A Gigabit Ethernet (en4) | 192.168.50.147 |
| Buffer Router | Linksys E2500 | 192.168.50.1 |
| Lab Gateway | Cisco 2600 Router | 192.168.50.10 |
| Streaming Server | External UDP host | 185.81.126.3 |

---

## Traffic Generated for This Lab

```bash
sudo arp -d -a                  # Flush ARP cache — forces fresh ARP exchange
ping -c 5 192.168.50.10         # 5 ICMP Echo Requests to Cisco 2600
ping -c 3 8.8.8.8               # 3 ICMP Echo Requests to Google DNS
dig google.com                  # DNS query (captured on en0, not en4)
dig cisco.com MX                # DNS MX record query
```

---

## Protocols Observed in Capture

| Protocol | Source | Notes |
|---|---|---|
| ARP | MacBook / Linksys / Cisco 2600 | Cache flush forced fresh exchange |
| ICMP | MacBook ↔ Cisco 2600 | 5 Echo Requests + 5 Echo Replies |
| UDP | MacBook ↔ 185.81.126.3 | Music streaming — port 1233 |
| IGMPv2 | Linksys (192.168.50.1) | Multicast group management |
| CDP | Cisco 2600 | Device discovery — revealed hostname `NetOps-R1.netops.lab` |

> **Passive Discovery:** The CDP packet revealed the Cisco 2600's configured hostname `NetOps-R1.netops.lab` without any CLI login. CDP broadcasts device identity, IOS version, and interface information every 60 seconds. This is passive reconnaissance from a packet capture — a real security consideration in production environments.

---

## Part 1 — Protocol Isolation Filters

The most fundamental filters — show only one protocol at a time.

| Filter | What It Shows | Result in This Capture |
|---|---|---|
| `arp` | All ARP traffic | Request/reply exchanges between MacBook, Linksys, Cisco 2600 |
| `icmp` | All ICMP traffic | 10 packets — 5 requests + 5 replies |
| `udp` | All UDP traffic | Streaming traffic + IGMPv2 multicast |
| `cdp` | Cisco Discovery Protocol | Cisco 2600 device announcements |

**Key observation:** The filter bar turns **green** for valid filters and **red** for invalid syntax. Green = Wireshark understands the filter. Red = check your syntax before running.

---

## Part 2 — Directional IP Filters

Three filters that look similar but produce different results:

### `ip.addr == 192.168.50.10`
Shows traffic where `192.168.50.10` is **either** source **or** destination — both directions of the conversation.

### `ip.src == 192.168.50.147`
Shows only traffic **from** the MacBook — outbound only. Cisco 2600 replies disappear.

### `ip.dst == 192.168.50.10`
Shows only traffic **to** the Cisco 2600 — inbound to the router only.

**The distinction:**

```
ip.addr  == x.x.x.x    →  bidirectional  (to OR from)
ip.src   == x.x.x.x    →  outbound only  (from this host)
ip.dst   == x.x.x.x    →  inbound only   (to this host)
```

---

## Part 3 — Logical Operators

### AND operator (`&&` or `and`)

Requires **both** conditions to be true simultaneously:

```
ip.src == 192.168.50.147 && icmp
```

Result: Only ICMP packets sent by the MacBook — 5 Echo Requests. No replies, no UDP.

```
ip.src == 192.168.50.147 and icmp
```

Identical result — `&&` and `and` are interchangeable in Wireshark.

### NOT operator (`!` or `not`)

Excludes matching packets:

```
!arp               # Hide all ARP traffic
!udp               # Hide all UDP traffic
not arp and not dns  # Exclude both ARP and DNS
```

`!` and `not` are interchangeable — both produce identical results. `!` is faster to type, `not` is more readable.

### Chaining exclusions to strip noise

```
!arp && !udp && !igmp && !cdp
```

Result: 10 packets — only ICMP traffic. Stripped ARP broadcasts, UDP streaming, IGMPv2 multicast, and CDP announcements from the entire capture in one filter.

**This is the noise reduction technique.** In production captures with tens of thousands of packets, chained exclusions isolate the relevant traffic in seconds.

---

## Part 4 — Right-Click Filter Creation

The most efficient way to build filters — no memorization required.

**Procedure:**
1. Click any packet in the capture list
2. Expand the relevant protocol section in the Packet Detail pane
3. Right-click any field
4. Select **Apply as Filter → Selected** or **Prepare as Filter → Selected**

**Apply as Filter** — immediately applies the filter to the capture  
**Prepare as Filter** — writes the filter in the bar without applying — allows further editing before running

### Example — ICMP Type Filter

Right-clicked **Type: 8 (Echo Request)** in the Internet Control Message Protocol section.

Wireshark wrote: `icmp.type == 8`

Result: Only 5 packets — outbound Echo Requests only. Replies (type 0) excluded automatically.

### Example — EtherType Filter (intentional experiment)

Right-clicked **Type: IPv4 (0x0800)** in the Ethernet II section.

Wireshark wrote: `eth.type == 0x0800`

Result: All IPv4 traffic — UDP and ICMP both visible. ARP (`eth.type == 0x0806`) excluded because ARP is not IPv4.

> **Why this matters:** `eth.type == 0x0800` is a valid filter for isolating all IPv4 traffic while excluding Layer 2 only frames. Different field, different scope, both useful.

**The right-click technique works on any field in any protocol.** You never need to memorize exact field names — right-click what you see in the decode and Wireshark writes the syntax.

---

## Part 5 — Saved Filter Library

Filters saved to the Wireshark bookmark library during this lab:

| Filter Name | Expression | Purpose |
|---|---|---|
| ICMP Echo Request Only | `icmp.type == 8` | Outbound pings only |
| IPv4 Traffic Only | `eth.type == 0x0800` | All IPv4, excludes ARP/non-IP frames |

Saved via: bookmark icon (left of filter bar) → **Save this filter** → name it.

Saved filters appear in the bookmark dropdown for instant reuse across all future captures.

---

## Unexpected Discoveries

### IGMPv2 Multicast Traffic
Applying `!udp` revealed IGMPv2 traffic from the Linksys router (`192.168.50.1`) to multicast group addresses:

| Destination | Group | Purpose |
|---|---|---|
| 224.0.0.1 | All Hosts | Every multicast-capable device |
| 224.0.0.2 | All Routers | Only routers on the segment |
| 239.255.255.250 | UPnP/SSDP | Universal Plug and Play devices |

This traffic was invisible until UDP was excluded. **Filtering reveals what noise hides.**

### CDP Device Fingerprinting
The CDP frame from `Cisco_05:c2:61` revealed:
- Device hostname: `NetOps-R1.netops.lab`
- Protocol bundle: CDP/VTP/DTP/PAgP in a single frame
- Sent to Cisco multicast MAC: `01:00:0c:cc:cc:cc`

Hostname and device identity obtained without any CLI access — passive capture only.

---

## Key Takeaways

| Concept | What the Lab Proved |
|---|---|
| Display filters are non-destructive | All packets remain in capture — only display changes |
| `ip.addr` vs `ip.src` vs `ip.dst` | Same IP, three different scopes — bidirectional vs directional |
| `&&` and `and` are identical | Two syntaxes, same result — use whichever reads cleaner |
| `!` and `not` are identical | Same operator, two forms |
| Chained exclusions strip noise | `!arp && !udp && !igmp && !cdp` → 10 packets from thousands |
| Right-click builds filters automatically | No field name memorization required |
| Filtering reveals hidden protocols | IGMPv2 and CDP only visible after excluding dominant traffic |
| Passive capture can fingerprint devices | CDP revealed router hostname without login |

---

## Real-World Relevance

- **NOC troubleshooting:** `!arp && !stp` is the first filter applied in most NOC captures — removes Layer 2 noise to expose routed traffic problems
- **ACL verification:** `tcp.flags.reset == 1` isolates RST packets that prove an ACL is blocking connections
- **Security analysis:** CDP and LLDP filters reveal network topology from passive capture — reason Cisco recommends disabling CDP on edge ports
- **Interview answer:** "I use display filters to isolate specific conversations from large captures — protocol filters for noise reduction, directional filters for source/destination analysis, and right-click creation for field-specific queries"

---

## Filter Reference — Used in This Lab

```
arp                                      # ARP only
icmp                                     # ICMP only
udp                                      # UDP only
ip.addr == 192.168.50.10                 # Bidirectional — to or from host
ip.src == 192.168.50.147                 # Outbound from MacBook only
ip.dst == 192.168.50.10                  # Inbound to Cisco 2600 only
ip.src == 192.168.50.147 && icmp         # MacBook ICMP outbound only
!arp                                     # Exclude ARP
!udp                                     # Exclude UDP
not arp and not dns                      # Exclude ARP and DNS
!arp && !udp && !igmp && !cdp            # Strip all noise — ICMP only
icmp.type == 8                           # Echo Request only (right-click generated)
eth.type == 0x0800                       # IPv4 frames only (right-click generated)
```

---

*Lavoisier Cornerstone — [lavoisier.dev](https://lavoisier.dev) | [github.com/cornerstonian](https://github.com/cornerstonian)*  
*Part of the [ccna-wireshark-labs](https://github.com/cornerstonian/wireshark-traffic-analysis-ccna) project series*
