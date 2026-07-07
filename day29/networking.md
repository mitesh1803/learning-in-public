![Progress](https://img.shields.io/badge/Progress-29%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 29 — Networking Fundamentals for DevOps

## 📝 Topic: IP Addressing, Subnets, CIDR, Ports & the OSI Model
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 07, 2026

---

## 🎯 Learning Objectives
* Understand what an IP address is and how IPv4's 4-byte structure works.
* Understand why subnets exist and how they enforce isolation and security.
* Learn CIDR notation and calculate available host addresses from a suffix.
* Understand ports as the mechanism for identifying a specific service on a machine.
* Understand DNS resolution and the TCP three-way handshake as prerequisites to any data transfer.
* Walk through all 7 layers of the OSI model, in order, for both transmission and receipt.
* Understand how the TCP/IP model simplifies the OSI model in modern practice.

---

## 🔢 Part 1 — IP Addresses

### Purpose

```
Every device connected to a network gets a UNIQUE identifier
→ enables tracking, monitoring, and access control
→ without it, there's no way to know which device sent/received what
```

### IPv4 Structure

```
IPv4 address = 4 bytes (32 bits total)
Written as four decimal numbers separated by dots:

  192 . 168 . 1 . 1
   ↑     ↑    ↑   ↑
 octet  octet octet octet   (each = 8 bits)
```

```
Each octet: 8 bits → 2^8 = 256 possible values → range 0-255

Why the range is 0-255, not 1-256:
  8 bits can represent values 00000000 through 11111111
  = 0 through 255 in decimal (256 total values, starting at 0)
```

---

## 🧩 Part 2 — Subnets

### Purpose

```
A subnet = a segment of a larger network
Used specifically for: SECURITY, PRIVACY, ISOLATION
```

### Why Segmentation Matters

```
Example: A company network split into subnets

  Subnet A: "Finance"        → sensitive DB servers, restricted access
  Subnet B: "Free/General"   → general employee devices, less restricted

If a device in Subnet B gets compromised:
  → Finance subnet remains isolated and unaffected
  → Blast radius of the compromise is contained to Subnet B
```

### Subnet Types

| Type | Characteristic |
|---|---|
| **Public Subnet** | Has internet access, typically via an Internet Gateway |
| **Private Subnet** | Isolated from the internet entirely — no direct inbound/outbound public traffic |

> **Practical takeaway:** databases and internal services almost always belong in a private subnet; load balancers and public-facing web servers belong in a public subnet.

---

## 📐 Part 3 — CIDR (Classless Inter-Domain Routing)

### Purpose

```
CIDR notation defines the SIZE of a network or subnet —
i.e., how many IP addresses it actually contains.

Written as: <network-address>/<suffix>
  e.g., 10.0.0.0/24
```

### The Calculation Rule

```
IPv4 total bits = 32

Host bits available = 32 - <CIDR suffix>
Number of IPs = 2^(host bits)
```

```
Example 1:  /24
  32 - 24 = 8 host bits remaining
  2^8 = 256 IP addresses

Example 2:  /27
  32 - 27 = 5 host bits remaining
  2^5 = 32 IP addresses
```

```
Rule of thumb:
  SMALLER suffix number  → MORE bits left → MORE IPs  → LARGER network
  LARGER suffix number   → FEWER bits left → FEWER IPs → SMALLER network
```

### Best Practice: Private IP Ranges

```
When creating private subnets, always use standardized
private IP ranges to avoid conflicting with public internet addresses:

  10.0.0.0    – 10.255.255.255
  172.16.0.0  – 172.31.255.255
  192.168.0.0 – 192.168.255.255
```

---

## 🔌 Part 4 — Ports

### IP vs. Port

```
IP address    → identifies the MACHINE on the network
Port number   → identifies the SPECIFIC SERVICE/APPLICATION
                running on that machine

Together:  172.16.3.4:9191
              ↑          ↑
          machine    specific app/service listening here
```

### Reserved / Well-Known Ports

```
Many standard services default to specific, widely recognized ports:

  3306  → MySQL
  8080  → Jenkins (commonly)
  22    → SSH
  80    → HTTP
  443   → HTTPS
```

> **Best practice:** avoid deploying custom applications on these well-known ports — collisions cause confusing conflicts, and using non-standard ports for custom apps keeps intent clear for anyone reading a Security Group or firewall rule later.

---

## 🌐 Part 5 — Understanding the OSI Model

### Pre-Requisites Before Any Layer Even Engages

```
Before OSI layers process anything, two things happen first:

1. DNS Resolution
   → Router maps a domain name (google.com) to an IP address
   → Checks LOCAL CACHE first
   → Falls back to the ISP's DNS server if not cached

2. TCP Handshake (three-way)
   → SYN       (client → server: "I want to connect")
   → SYN-ACK   (server → client: "Acknowledged, I'm ready too")
   → ACK       (client → server: "Confirmed, connection established")
```

> **Why this matters:** neither DNS resolution nor the TCP handshake are part of the OSI model's 7 layers themselves — they're the setup steps that make the layered journey possible in the first place.

### The 7 Layers — Transmission Path (L7 → L1)

```
Layer 7 — Application
  → Browser initiates the request using HTTP/HTTPS

Layer 6 — Presentation
  → Handles data encryption and formatting
  → Essential for secure transmission (this is where HTTPS encryption applies)

Layer 5 — Session
  → Maintains a session between client and server
  → Avoids re-authenticating on every single interaction

  Note: Layers 7, 6, and 5 are primarily handled by the BROWSER itself,
  not by the network stack below it.

Layer 4 — Transport
  → SEGMENTATION: large data broken into smaller chunks
  → Protocol identified here: typically TCP or UDP

Layer 3 — Network
  → Routers add SOURCE and DESTINATION IP addresses to segments
  → Segments become PACKETS
  → Determines the optimal route across networks

Layer 2 — Data Link
  → Data converted into FRAMES
  → MAC addresses added so switches can route within the LOCAL network

Layer 1 — Physical
  → Frames converted into raw electronic signals (bits)
  → Transmitted across physical media (cables, fiber, etc.)
```

```
Full picture:

  Application data
        ↓ (L7 Application)
        ↓ (L6 Presentation — encrypt/format)
        ↓ (L5 Session — maintain session)
        ↓ (L4 Transport — segment, TCP/UDP)
        ↓ (L3 Network — add IPs, form Packets)
        ↓ (L2 Data Link — add MACs, form Frames)
        ↓ (L1 Physical — convert to signals, transmit)

  ═══════════════ physical wire/fiber ═══════════════

  Receiving server processes the EXACT REVERSE:
  L1 → L2 → L3 → L4 → L5 → L6 → L7
```

---

## 📊 Summary & Key Takeaways

```
Transmission direction:
  Sender:    L7 → L1  (top-down, packaging data for transmission)
  Receiver:  L1 → L7  (bottom-up, unpacking data back to the application)
```

### TCP/IP Model vs. OSI Model

```
OSI Model (7 layers, theoretical/academic reference):
  Application | Presentation | Session | Transport | Network | Data Link | Physical

TCP/IP Model (modern practical standard):
  Application (combines OSI's Application + Presentation + Session)
  Transport
  Internet   (≈ OSI Network)
  Link       (≈ OSI Data Link + Physical)
```

> **Why this matters:** in modern practice, engineers mostly reason in terms of the simpler TCP/IP model — the OSI model remains the standard *teaching* framework because its finer-grained layers make it easier to explain exactly where a specific function (encryption, sessions, routing, framing) happens.

### DevOps Perspective

> Understanding the OSI model is framed as a **"good to have"** for DevOps engineers — not something used explicitly day-to-day, but genuinely valuable when troubleshooting connectivity issues, since knowing which layer a failure is likely occurring at (DNS? TCP handshake? routing? physical link?) narrows down debugging significantly faster than guessing.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **IPv4 Address** | A 32-bit address written as 4 octets (0-255 each), uniquely identifying a device on a network |
| **Octet** | One of the four 8-bit segments making up an IPv4 address |
| **Subnet** | A segmented portion of a larger network, used for isolation and security |
| **Public Subnet** | A subnet with internet access, typically via an Internet Gateway |
| **Private Subnet** | A subnet isolated from direct internet access |
| **CIDR** | Notation (e.g. `/24`) defining network size via host bits: `2^(32 - suffix)` addresses |
| **Private IP Ranges** | Standardized ranges (10.x, 172.16-31.x, 192.168.x) reserved for internal networks |
| **Port** | A number identifying a specific service/application on a machine (e.g. `:9191`) |
| **Well-Known Ports** | Reserved ports for standard services (22=SSH, 80=HTTP, 3306=MySQL, etc.) |
| **DNS Resolution** | Mapping a domain name to an IP address, checked locally first, then via the ISP's DNS |
| **TCP Three-Way Handshake** | SYN → SYN-ACK → ACK sequence that establishes a TCP connection before data transfer |
| **OSI Model** | The 7-layer conceptual model (Application → Physical) describing how data travels across a network |
| **Segmentation** | Breaking large data into smaller chunks at Layer 4 (Transport) |
| **Packet** | Data with source/destination IP addresses added, formed at Layer 3 (Network) |
| **Frame** | Data with MAC addresses added, formed at Layer 2 (Data Link) |
| **MAC Address** | A hardware-level address used by switches to route data within a local network |
| **TCP/IP Model** | The modern, simplified 4-layer practical model that collapses OSI's top 3 layers into one Application layer |

---

## 📂 Summary of Tasks
- ✅ Understood: IPv4's 4-byte/32-bit structure and why each octet ranges 0-255.
- ✅ Understood: Why subnets exist — isolation, security, and containing compromise blast radius.
- ✅ Distinguished: Public subnets (internet-facing) vs. private subnets (isolated).
- ✅ Learned: CIDR notation and the `2^(32 - suffix)` calculation for available IPs.
- ✅ Noted: Standard private IP ranges (10.x, 172.16-31.x, 192.168.x) to avoid public IP conflicts.
- ✅ Understood: Ports as the service-level identifier layered on top of an IP address.
- ✅ Noted: Well-known reserved ports to avoid when deploying custom applications.
- ✅ Understood: DNS resolution and the TCP three-way handshake as pre-requisites before OSI layers engage.
- ✅ Walked through: All 7 OSI layers in transmission order (L7 → L1) and receipt order (L1 → L7).
- ✅ Understood: How the TCP/IP model collapses OSI's Application/Presentation/Session into one layer.
- ✅ Framed: OSI model knowledge as a valuable "good to have" for DevOps-level network troubleshooting.

---

## 💡 My Takeaway

The CIDR calculation (`2^(32 - suffix)`) is the piece I most needed a clean mental model for — I'd seen `/24` and `/16` all over Terraform VPC configs without a solid intuition for what they actually mean in terms of usable IP count. Having a repeatable formula instead of just memorizing "`/24` = 256 addresses" as a fact makes it easy to reason about any suffix on the fly, which matters directly for the Terraform VPC/subnet modules I built in the DevOps portfolio project.

The OSI model walkthrough reframed something I'd always treated as a memorization exercise (Please Do Not Throw Sausage Pizza Away, etc.) into an actual sequence of concrete transformations: encrypt/format → maintain session → segment → address+route → frame+MAC → physical signal. Tying each layer to a specific, tangible action (not just an abstract name) makes it far more useful for actual troubleshooting — if a connection fails, "which layer" becomes a real diagnostic question instead of trivia.

The subnet isolation example landed well too — thinking about a Finance vs. General subnet split as blast-radius containment is the same security principle I've seen show up in RBAC (least privilege) and Secrets management (encryption + access control) — it's clearly a recurring theme across this whole DevOps track: assume something WILL be compromised, and design so the damage stays contained.

---

## 🔗 Resources
* [IPv4 Addressing Explained (Cloudflare)](https://www.cloudflare.com/learning/network-layer/what-is-ipv4/)
* [CIDR Notation Explained (Cloudflare)](https://www.cloudflare.com/learning/network-layer/what-is-cidr-notation/)
* [OSI Model Explained (Cloudflare)](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-osi-model/)
* [TCP Three-Way Handshake (Cloudflare)](https://www.cloudflare.com/learning/ddos/glossary/tcp-ip/)
* [AWS VPC and Subnetting Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*