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
 

## Module 2: Network Components, Types, and Connections (cont'd) - Feb 5, 2026

### Peer-to-peer (P2P) networks (2.1.3)
- In P2P, devices can act as both client and server.
- Common in homes/small offices (file sharing, printer sharing).
- Smallest P2P can be two directly connected computers (wired or wireless).
- Larger P2P setups often use a switch to connect multiple devices.

**Pros:**
- Easy to set up
- Less complex
- Lower cost (may not require dedicated servers)
- Works for simple sharing tasks

**Cons:**
- No centralized administration
- Less secure
- Not scalable
- Performance can drop when hosts act as both client and server

**Key point:** Larger businesses usually use dedicated servers because of higher traffic and more service requests.

### Peer-to-peer applications (2.1.4)
- A P2P app allows each device to behave as both client and server in the same system.
- Often requires:
  - a user interface (front end)
  - a background service (running in the background)
- Some P2P apps use a hybrid model:
  - sharing is decentralized (peers share directly)
  - a centralized index server helps peers find where resources are located

### Multiple roles on a network (2.1.5)
- A single computer can run:
  - multiple server services (ex: file + web + email)
  - multiple client apps (ex: browser + email client + messaging)
- One host can connect to multiple servers at the same time (email + web + chat, etc.).

---

## Network infrastructure (2.2)

### Network symbols (2.2.1)
- Networking diagrams use standard symbols to represent common devices (routers, switches, firewalls, etc.).

### Network infrastructure overview (2.2.2)
Network infrastructure is the "platform" that enables communication. It includes three categories:

**End devices (hosts):**
- Desktop/laptop
- Printer
- IP phone
- Tablet
- TelePresence endpoint

**Intermediate devices:**
- Wireless router
- LAN switch
- Router
- Multilayer switch
- Firewall appliance

**Network media:**
- Wireless
- LAN media
- WAN media

**Key idea:** Some infrastructure is visible (cables, switches). Wireless media is not visible, signals travel through the air.

### End devices (2.2.3)
- End devices (hosts) are what users interact with most.
- They are either the source or destination of a network message.
- Hosts must be uniquely identified using addresses.

**Examples:**
- Computers (workstations, laptops, file/web servers)
- Network printers
- Phones and video conferencing equipment
- Security cameras
- Mobile devices (smartphones, tablets, scanners/readers)

---

## ISP connectivity options (2.3)

### What an ISP is (2.3.1)
- An ISP connects a home or business network to the internet.
- Examples: cable company, phone provider, cellular provider, or a provider leasing bandwidth.
- Many ISPs also offer services like:
  - email accounts
  - storage
  - web hosting
  - backup/security services
- ISPs connect to other ISPs in a hierarchy to move traffic efficiently.
- The internet backbone is made of high-speed links, mostly fiber-optic cable, including undersea cables.

### How home users connect (2.3.2)
- A direct modem-to-computer connection exists, but it is not recommended because the computer is exposed.
- The common setup is:
  - ISP connection → modem → router
  - router provides switching + Wi-Fi access point + basic security + internal IP addressing

### Cable vs DSL (2.3.3)

**Cable:**
- Uses coax cable (same cable used for TV).
- High bandwidth, always-on.
- Cable modem separates internet signal and provides Ethernet to a host/LAN.

**DSL (Digital Subscriber Line):**
- Uses telephone line.
- Always-on.
- Splits into channels:
  - voice
  - faster download
  - slower upload
- Speed depends on line quality and distance from the provider's central office.

### Other connectivity options (2.3.4)
- **Cellular:** available where there is cell service, often metered and performance depends on tower/device.
- **Satellite:** good where cable/DSL is not available, requires clear line of sight, higher setup cost, performance varies.
- **Dial-up:** low bandwidth, only for situations with no better option.
- Some urban areas may have direct fiber to apartments/offices.

---

## Module 2 summary (2.4)
- Hosts can be clients, servers, or both. P2P networks are common in small environments but are less secure and do not scale well.
- Network infrastructure consists of end devices, intermediate devices, and network media.
- ISPs connect local networks to the global internet. Common home connections are cable or DSL, with other options like cellular and satellite depending on location.

