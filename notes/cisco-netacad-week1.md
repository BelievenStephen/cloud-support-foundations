# Cisco NetAcad Networking Basics, Week 1 Notes (Feb 4, 2026)

## Goal (course)
Get an introductory, practical view of how networks and the internet work.

## Key outcomes (what this course aims to teach)
- Core network communication concepts (clients/servers, protocols, media)
- Network types (home, SOHO, enterprise, internet)
- How devices connect (wired, Wi-Fi)
- Ethernet basics
- IP addressing basics (IPv4 + IPv6)
- DHCP basics
- Routers connecting networks
- ARP basics
- Basic connectivity testing and troubleshooting tools

---

## Module 1: Communications in a Connected World

### What the internet is
- The internet is a worldwide collection of interconnected networks (“network of networks”).
- Online destinations (websites, email, games) live on servers connected to local networks that reach the global internet.

### Network types (high level)
- **Home networks:** small, a few devices
- **SOHO networks:** small office/home office, share resources (printers/files) + shared internet
- **Medium/Large enterprise networks:** many users/devices, built for business operations
- **Worldwide networks:** large-scale interconnection of many networks (the internet)

### Connected devices
- **Mobile:** phone, tablet, smartwatch, smart glasses
- **Connected home:** security system, appliances, smart TV, gaming console
- **Other:** smart cars, RFID, sensors/actuators, medical devices

### Personal data types (privacy concept)
- **Volunteered data:** you intentionally provide it (forms, profiles)
- **Observed data:** collected by tracking actions (location data, activity logs)
- **Inferred data:** derived from analysis (ex: credit score)

### Bits, bytes, and encoding
- Networks transmit data as **bits** (0s and 1s).
- **Byte = 8 bits.**
- **ASCII** is one example encoding where characters map to 8-bit patterns.

### How bits move across a network (signals + media)
Data becomes signals that travel across network media:
- **Electrical (copper):** electrical pulses
- **Optical (fiber):** light pulses
- **Wireless:** radio/microwave/infrared through the air

Home/small business commonly uses copper + Wi-Fi. Larger networks use fiber for long distance and reliability.

### Bandwidth vs throughput vs latency
- **Bandwidth:** theoretical capacity of a link (Kbps, Mbps, Gbps, Tbps)
- **Throughput:** actual delivered rate in real conditions (often lower than bandwidth)
- **Latency:** time delay for data to travel end-to-end

Why throughput is lower than bandwidth:
- other traffic on the link
- type/size of data
- number of network devices between endpoints (adds latency)
- the slowest link on the path limits end-to-end throughput

---

## Module 2: Network Components, Types, and Connections (start)

### Clients and servers (roles)
- **Host:** any device that sends/receives network messages
- **Server:** provides services/resources (web, email, files)
- **Client:** requests and displays services/resources (browser, email client, file explorer)

Examples:
- **Web:** browser (client) requests pages from web server (server)
- **Email:** email client (client) connects to email server (server)
- **File:** file explorer (client) accesses file server (server)

Key idea: a computer can act as a client, server, or both depending on the software running.

---

## What I want to connect to my labs (next)
- Practice identifying whether a problem is:
  - service down vs port not listening vs DNS vs routing
- Use basic tools regularly:
  - `dig`, `curl`, `ss`, `ip route`, `traceroute`
