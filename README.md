# 🦈 Lab 2 — Wireshark & Network Analysis

![Wireshark](https://img.shields.io/badge/Wireshark-Free%20%26%20Open%20Source-1679A7?style=flat&logo=wireshark&logoColor=white)
![Azure](https://img.shields.io/badge/Platform-Local%20or%20Azure%20VM-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![Cost](https://img.shields.io/badge/Cost-%240-brightgreen?style=flat)
![Status](https://img.shields.io/badge/Status-Complete-success?style=flat)

> Captured and analysed live network traffic using Wireshark — inspecting DNS queries, TCP three-way handshakes, cleartext HTTP credentials, and full TCP stream reconstruction.

---

## 📋 Lab Summary

| Field | Value |
|-------|-------|
| **Certification Alignment** | CompTIA Network+ · Security+ · CySA+ |
| **Tools Used** | Wireshark (free, open source) · nslookup · tshark |
| **Estimated Cost** | $0 — Wireshark is permanently free, no account required |
| **Time to Complete** | 2–4 hours across multiple sessions |
| **Career Relevance** | SOC Analyst · Network Engineer · Cloud Security Engineer · Incident Responder |

---

## 🏗️ Architecture — How Wireshark Captures Traffic

```
┌─────────────────────────────────────────┐
│              Internet                   │
│  Web Servers · DNS Servers · Remote     │
│  DNS:53 · HTTP:80 · HTTPS:443 · ICMP   │
└──────────────────┬──────────────────────┘
                   │ all frames
                   ▼
┌─────────────────────────────────────────┐
│           Router / Switch               │
│  Home network — forwards all frames     │
└──────────────────┬──────────────────────┘
                   │ raw packets
                   ▼
┌─────────────────────────────────────────┐
│       Network Interface Card (NIC)      │
│  Promiscuous mode — captures ALL        │
│  packets, not just packets to this host │
└──────────────────┬──────────────────────┘
                   │ decoded frames
                   ▼
┌─────────────────────────────────────────┐
│              Wireshark                  │
│  Decodes packets · applies filters      │
│  reassembles streams · exports .pcapng  │
└──────┬──────────┬──────────┬────────────┘
       │          │          │
    Capture    Filter     Analyse
   live/file  dns/tcp/ip  streams
```

**Data flow:** Raw network frames → NIC in promiscuous mode → Wireshark decodes every layer from Ethernet frame to application payload → display filters isolate the traffic you care about → export as `.pcapng` for portfolio evidence.

---

## ✅ What I Built

- **Live traffic capture** on a network interface using Wireshark in promiscuous mode
- **DNS capture and analysis** — isolated query and response packets, verified A record resolution
- **TCP three-way handshake observation** — identified SYN, SYN-ACK, and ACK packets in sequence
- **Cleartext credential extraction** — demonstrated HTTP POST request exposing username/password in plaintext
- **TCP stream reconstruction** — followed a full client-server conversation from individual packet fragments
- **Saved .pcapng files** for DNS lookup, TCP handshake, and HTTP stream (see `/captures` folder)

---

## 🔑 Key Concepts

| Concept | Description |
|---------|-------------|
| **Packet** | A small unit of data with a header (src IP, dst IP, port) and payload. Wireshark shows each one individually. |
| **Promiscuous mode** | NIC captures every packet on the segment — not just packets addressed to your machine |
| **TCP three-way handshake** | SYN → SYN-ACK → ACK. The connection setup before any data is exchanged |
| **DNS A record** | Maps a domain name to an IPv4 address. Type 1 in the packet. Happens before every website visit |
| **HTTP vs HTTPS** | HTTP sends data in cleartext — credentials visible in packets. HTTPS encrypts with TLS |
| **Display filters** | Applied after capture to show only matching traffic without discarding the rest |

---

## 🔎 Essential Display Filters

```wireshark
dns                          # All DNS queries and responses
http                         # Unencrypted HTTP traffic only
tcp                          # All TCP traffic
tcp.flags.syn == 1           # TCP SYN packets — connection attempts only
tcp.flags.reset == 1         # TCP RST packets — refused or closed connections
icmp                         # All ICMP traffic including ping
ip.addr == 192.168.1.1       # All traffic to or from a specific IP
ip.src == 10.0.0.5           # Traffic from a specific source IP only
tcp.port == 443              # All HTTPS traffic by port
http.request.method == POST  # HTTP POST requests — form submissions, logins
```

> Display filters are applied after capturing — they let you look at the same capture through different lenses without re-capturing. Apply `dns`, examine it, remove it, apply `tcp` — the full capture is always underneath.

---

## 🧪 Exercises Completed

### Exercise A — DNS Lookup Capture

Triggered a DNS query using `nslookup google.com` while Wireshark was capturing. Applied the `dns` display filter and identified:

- The **query packet** — "Standard query A google.com" in the Info column
- The **response packet** — containing the A record with Google's IP address
- Verified the returned IP matched what `nslookup` showed in the terminal

```bash
# Command run in terminal to generate the DNS traffic
nslookup google.com
```

📸 *See `/screenshots/03-dns-query-packet.png` and `/screenshots/04-dns-response-a-record.png`*

**Real-world relevance:** Unexpected DNS queries to unusual domains in a capture are often the first indicator of malware communicating with a command and control server.

---

### Exercise B — TCP Three-Way Handshake

Navigated to `http://example.com` and applied `tcp and ip.addr == [example.com IP]` to isolate the connection. Identified all three packets in sequence:

| Packet | Flags | Meaning |
|--------|-------|---------|
| 1st | SYN | Client: I want to connect. Here is my sequence number. |
| 2nd | SYN, ACK | Server: Request received. Connection accepted. |
| 3rd | ACK | Client: Confirmed. Ready to send data. |

📸 *See `/screenshots/05-tcp-syn-synack-ack.png`*

**Real-world relevance:** SYN with no SYN-ACK = connection refused or server unreachable. RST packet = connection forcibly closed. These two patterns are the most common findings when diagnosing connectivity failures.

---

### Exercise C — Cleartext Credentials in HTTP

Submitted a test login form over HTTP and applied `http.request.method == POST`. Located the POST packet, expanded the "HTML Form URL Encoded" layer in the packet detail pane, and observed the username and password in plaintext.

📸 *See `/screenshots/06-http-post-credentials.png`*

> **Educational note:** This exercise demonstrates exactly why HTTPS is mandatory for any login form. Without TLS, credentials are readable by anyone on the network path — ISP, coffee shop router, or man-in-the-middle attacker.

---

### Exercise D — TCP Stream Reconstruction

Right-clicked an HTTP packet → **Follow → TCP Stream**. Wireshark reassembled all packets from that connection into a readable conversation:

- Red text = browser's request to the server
- Blue text = server's response back to the browser

📸 *See `/screenshots/07-tcp-stream-follow.png`*

**Real-world relevance:** Individual packets are fragments. Stream view shows the complete conversation — what data was transferred, what commands were sent, what the server responded with. This is how incident responders reconstruct network events during an investigation.

---

## 📁 Repository Structure

```
homelab-wireshark-network-analysis/
│
├── README.md
├── captures/
│   ├── 01-dns-lookup-google.pcapng          # Exercise A — DNS query and response
│   ├── 02-tcp-three-way-handshake.pcapng    # Exercise B — SYN SYN-ACK ACK sequence
│   └── 03-http-stream-follow.pcapng         # Exercise D — full TCP stream reconstruction
└── screenshots/
    ├── 01-wireshark-interface.png            # Wireshark welcome screen with interfaces
    ├── 02-dns-filter-applied.png             # dns display filter active in filter bar
    ├── 03-dns-query-packet.png               # DNS query packet selected and expanded
    ├── 04-dns-response-a-record.png          # DNS response showing A record IP
    ├── 05-tcp-syn-synack-ack.png             # Three packets: SYN → SYN-ACK → ACK
    ├── 06-http-post-credentials.png          # POST packet with cleartext credentials visible
    └── 07-tcp-stream-follow.png              # Full TCP stream reconstructed in stream view
```

> The `.pcapng` files in `/captures` are actual captured network traffic — they can be downloaded and opened in Wireshark for independent verification.

---

## 💾 Saving and Exporting Captures

```bash
# Save full capture
File → Save As → .pcapng format

# Export only packets matching current display filter
File → Export Specified Packets → Displayed

# Command-line capture with tshark (included with Wireshark)
tshark -i eth0 -w capture.pcapng -c 1000
# -i: interface   -w: output file   -c: stop after N packets
```

---

## ✔️ Verification Checklist

| Skill | Verified |
|-------|----------|
| Apply `dns` filter and identify matching query and response packets | ✅ |
| Identify SYN, SYN-ACK, ACK packets in a TCP handshake | ✅ |
| Apply `ip.addr`, `tcp.port`, and `http` filters from memory | ✅ |
| Follow a TCP stream and read the full HTTP request/response | ✅ |
| Save a capture, close Wireshark, reopen and confirm all packets present | ✅ |

---

*Part of an ongoing cybersecurity home lab series targeting SOC Analyst and Cloud Security Engineer roles.*
