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


## Module 3: Wireless and Mobile Networks - Feb 6, 2026

### Module objective
Configure a mobile device for wireless access and understand common wireless network types.

### Wireless network types used by mobile devices

**Cellular (GSM, 3G/4G/5G):**
- Uses towers and radio waves for voice, text, and data.
- Newer generations (4G/5G) are optimized for higher-speed data.

**GPS:**
- Uses satellites so a phone can calculate location (often within about 10 meters).

**Wi-Fi:**
- Connects to local networks through a wireless access point within range.
- Public Wi-Fi areas are "hotspots."

**Bluetooth:**
- Short-range, low-power wireless for accessories and device-to-device links.
- Supports multiple devices at once (up to ~8 connections).

**NFC:**
- Very short-range communication (a few centimeters).
- Common for tap-to-pay and quick device pairing.

### Mobile devices and Wi-Fi (practical points)

Use Wi-Fi when available to:
- avoid cellular data usage
- reduce battery use (Wi-Fi radio often uses less power than cellular)

**Basic Wi-Fi security practices:**
- avoid sending passwords in plaintext
- use a VPN when sending sensitive data on public Wi-Fi
- secure home Wi-Fi
- use WPA2 or higher encryption

### Wi-Fi connection behavior on phones
- Turn on Wi-Fi → device scans and lists nearby networks → select SSID → enter password if required.
- Devices typically auto-connect to known networks.
- If Wi-Fi is out of range, the device falls back to cellular data (if enabled).

### Manual Wi-Fi configuration (when auto-connect fails)

**Common reasons:**
- SSID broadcast is disabled (hidden network)
- device is not set to auto-join

**Key terms:**
- **SSID:** network name
- **Passphrase:** Wi-Fi password

**Important detail:** SSID and passphrase must match exactly.

### Cellular data settings (high level)
- Most users rely on cellular data only when Wi-Fi is unavailable.
- Phones switch between cellular generations (ex: 4G to 3G) as coverage changes, usually without obvious interruption.

### Bluetooth basics and pairing

Bluetooth is useful when cables are not practical (headsets, speakers, peripherals).

**Common uses:**
- hands-free headset
- keyboard/mouse
- car stereo/speakerphone
- tethering (sharing a connection)
- portable speakers

**Pairing process:**
- enable Bluetooth on both devices
- set accessory to discoverable/visible
- scan/select device
- enter a PIN/passkey if prompted (often stored for future auto-connect)

**Discoverable devices can advertise:**
- name, class, available services, supported features/spec versions

---

## Module 4: Build a Home Network (start)

### Module objective
Configure an integrated wireless router and wireless client to connect securely to the internet.

### Home network basics
- A home network can include PCs, consoles, smart TVs, printers/scanners, cameras, phones, and smart-home devices.

**Typical home router ports:**
- **LAN/Ethernet ports:** connect internal devices to the same local network (built-in switch)
- **Internet/WAN port:** connects the router to the ISP/modem and a different network (the outside network)

Many home routers also include a built-in wireless access point. Wireless clients usually join the same LAN as wired clients.

### Network technologies in the home
- Common Wi-Fi frequencies: 2.4 GHz and 5 GHz (unlicensed bands).
- Bluetooth also uses 2.4 GHz, but is designed for short-range, lower-speed connections.
- Wi-Fi (802.11) uses higher power than Bluetooth, giving better range and throughput.

### Wired network technologies (overview)

Ethernet is the most common wired LAN approach.

**Common media:**
- **Cat 5e (UTP):** 4 twisted pairs to reduce interference
- **Coax:** inner conductor + insulating layer + shield + jacket
- **Fiber optic:** glass/plastic strands, very high bandwidth, long distance

### Wireless standards and Wi-Fi
- IEEE creates wireless standards. 802.11 governs WLAN (Wi-Fi).
- Wi-Fi Alliance tests interoperability. Wi-Fi logo implies devices should work together.

### Common wireless router settings

**Network mode:** 802.11b/g/n or mixed mode

**SSID (network name):** case-sensitive, up to 32 characters

**Channel:** auto or manual

**SSID broadcast:** whether the network name is advertised

**Notes:**
- Mixed mode helps older devices connect.
- Disabling SSID broadcast does not secure a network by itself. Strong encryption is still required.

---

## Module 4: Build a Home Network (cont'd) - Feb 8, 2026

### Set Up a Home Router (4.4)

#### First-time setup (basic process)

- Many home routers include an automatic setup utility.
- Initial setup often uses a wired connection from a PC/laptop to a LAN/Ethernet port on the router (not the Internet/WAN port).
- The Internet/WAN port connects the router to the modem/ISP network.
- Some routers have a built-in modem. Connection types:
  - Cable often uses coax.
  - DSL often uses an RJ-11 phone-style cable.
- After connecting, the computer needs an IP address:
  - Usually provided automatically by the router's built-in DHCP server.
  - If no IP is received, you may need to configure: IP address, subnet mask, default gateway, DNS (per router docs).

#### Design considerations (before configuring)

**SSID naming:**
- If SSID broadcast is enabled, nearby devices can see the network name.
- Avoid including router brand/model in the SSID since it can reveal info that helps attackers find defaults/known weaknesses.

**Device compatibility and Wi-Fi standards:**
- Devices support different 802.11 standards (ex: b/g/n/ac).
- If the router is set to only allow newer standards, older devices may not connect.
- "Legacy mode" or "mixed mode" supports older devices but may reduce maximum performance.

**Adding new devices and access control:**
- Decide who should be able to join your network.
- Many routers support guest networks/guest SSIDs that allow internet access but restrict access to your internal LAN.

### Module 4 summary (4.5)

**What I learned:**

- Home networks typically connect a private home LAN to the public ISP network through a router (often with both wired + wireless).
- Common home Wi-Fi uses unlicensed 2.4 GHz and 5 GHz bands.
  - Bluetooth uses 2.4 GHz and is designed for short-range, low-power connections.
  - Wi-Fi (802.11) generally uses higher power for better range/throughput.
- Wired networking is still useful for stable performance. Common media:
  - Ethernet over Cat5e (UTP) is common for LANs.
  - Other options include coax and fiber, plus alternatives like powerline adapters where Ethernet wiring is not present.
- Wi-Fi standards:
  - IEEE 802.11 defines WLAN (Wi-Fi) standards.
  - Wi-Fi Alliance tests interoperability between vendors.
- Common router settings:
  - Network mode (b/g/n/ac or mixed)
  - SSID (network name, case-sensitive, up to 32 chars)
  - Channel (auto/manual)
  - SSID broadcast (whether the SSID is advertised)
- Security and usability tradeoffs:
  - Mixed/legacy modes help old devices connect but can reduce performance.
  - SSID broadcast off does not equal secure. Use strong security settings and control access (guest SSID as needed).


