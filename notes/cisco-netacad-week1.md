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

---

## Feb 12, 2026

## Module 5: Communication Principles

### Module objective
Explain the importance of standards and protocols in network communications.

**Topics covered:**
- Communication Protocols
- Communication Standards
- Network Communication Models

---

### Communication protocols basics

**Human communication analogy:**
- Before communicating, we establish rules/agreements
- Different expectations for different contexts (casual chat vs job interview)
- Must follow protocols for successful delivery and understanding

**Rules that govern human communication:**
- Identified sender and receiver
- Agreed-upon method of communicating (face-to-face, telephone, letter, photograph)
- Common language and grammar
- Speed and timing of delivery
- Confirmation or acknowledgment requirements

**Key insight:** Network communications share these fundamentals with human conversations.

---

### Why protocols matter

**Core concept:**
- Like humans, computers use rules (protocols) to communicate
- Local network = area where all hosts "speak the same language" (share common protocol)
- Without shared protocols, devices cannot communicate (like people speaking different languages in same room)

---

### Protocol characteristics

Networking protocols define many aspects of communication over the local network:

| Protocol Characteristic | Description |
|------------------------|-------------|
| **Message Format** | Specific format/structure required. Format depends on message type and channel used. |
| **Message Size** | Rules governing size of pieces are strict. May vary by channel. Long messages may need to be broken into smaller pieces for reliable delivery. |
| **Timing** | Determines: (1) speed of bit transmission, (2) when individual host can send data, (3) total amount of data in any one transmission. |
| **Encoding** | Messages converted to bits by sending host. Bits encoded into patterns of sounds, light waves, or electrical impulses (depending on media). Destination host receives and decodes signals. |
| **Encapsulation** | Process of adding header information to data pieces. Header contains addressing info (source and destination hosts). May include other info to ensure delivery to correct application. |
| **Message Pattern** | Some messages require acknowledgment before next message (request/response pattern). Others may stream across network without concern for delivery confirmation. |

---

### Communication standards

**Why standards matter:**
- With increasing devices and technologies, standards ensure reliable service delivery
- Standards = set of rules that determines how something must be done
- Networking and internet standards ensure all devices implement same rules/protocols in same manner

**Example benefit:**
- Different device types can send information to each other
- Email formatted, forwarded, and received according to standard
- PC can send email that mobile phone receives and reads (as long as both use same standards)

---

### Network standards organizations

**Standards development process:**
- Result of comprehensive cycle: discussion, problem solving, testing
- Different organizations develop, publish, and maintain standards

**Key organization:**
- **IETF (Internet Engineering Task Force):** Publishes and manages internet standards
- **RFC (Request for Comments):** Numbered documents tracking evolution of standards
- Each stage of development/approval is recorded in RFCs

---

### Network communication models (layered models)

**Why layered models are useful:**
- Visualize how protocols work together for network communications
- Show operation of protocols within each layer
- Show interaction between layers

**Benefits of layered models:**
- Assists in protocol design (defined info and interfaces per layer)
- Fosters competition (products from different vendors can work together)
- Enables technology changes at one level without affecting others
- Provides common language for describing networking functions

---

### The TCP/IP model

**History:**
- First layered model created in early 1970s (internet model)
- Defines four categories of functions for successful communications
- Suite of TCP/IP protocols follows this model structure
- Commonly referred to as TCP/IP Model

**Four layers:**

| TCP/IP Model Layer | Description |
|-------------------|-------------|
| **Application** | Represents data to the user, plus encoding and dialog control |
| **Transport** | Supports communication between various devices across diverse networks |
| **Internet** | Determines the best path through the network |
| **Network Access** | Controls the hardware devices and media that make up the network |

**Key concept:** Each layer has specific responsibilities and interacts with layers above and below it.

---

### Module 5 takeaways (so far)

**Core concepts learned:**
- Communication requires agreed-upon rules (protocols)
- Protocols define: message format, size, timing, encoding, encapsulation, and patterns
- Standards ensure different devices can communicate
- Layered models help visualize and design network communications
- TCP/IP model has four layers, each with specific functions

---

---

## Feb 16, 2026

### OSI Reference Model (5.3.4)

**Protocol model vs Reference model:**

**Protocol model:**
- Closely matches the structure of a particular protocol suite
- Describes functions that occur at each layer of protocols within the suite
- Example: TCP/IP model (describes TCP/IP suite functions)

**Reference model:**
- Describes functions that must be completed at a particular layer
- Does not specify exactly how a function should be accomplished
- Purpose: Aid in clearer understanding of functions and processes necessary for network communications
- Example: OSI model

**OSI Model:**
- Created by the Open Systems Interconnection (OSI) project at ISO
- Used for data network design, operation specifications, and troubleshooting

**OSI Model layers:**

| Layer | Name | Description |
|-------|------|-------------|
| 7 | Application | Contains protocols used for process-to-process communications |
| 6 | Presentation | Provides common representation of data transferred between application layer services |
| 5 | Session | Provides services to organize dialogue and manage data exchange |
| 4 | Transport | Defines services to segment, transfer, and reassemble data for individual communications |
| 3 | Network | Provides services to exchange individual pieces of data over the network between identified end devices |
| 2 | Data Link | Describes methods for exchanging data frames between devices over common media |
| 1 | Physical | Describes mechanical, electrical, functional, and procedural means to activate, maintain, and deactivate physical connections for bit transmission |

---

### OSI Model and TCP/IP Model Comparison (5.3.5)

**Why learn both models?**

**TCP/IP model:**
- Method of visualizing interactions of protocols in the TCP/IP protocol suite
- Describes networking functions specific to TCP/IP protocols
- Does not describe general functions necessary for all networking communications
- Example: Network access layer does not specify which protocols to use for physical transmission or signal encoding method

**OSI model:**
- Provides more granular layer breakdown
- OSI Layers 1 and 2 (Physical and Data Link) discuss procedures to access media and physical means to send data
- OSI model further divides TCP/IP's network access layer and application layer to describe discrete functions

**Layer mapping:**
- TCP/IP Internet layer = OSI Network layer (Layer 3)
- TCP/IP Transport layer = OSI Transport layer (Layer 4)
- TCP/IP Application layer maps to OSI Layers 5, 6, and 7
- TCP/IP Network Access layer maps to OSI Layers 1 and 2

---

### Module 5 Summary (5.4.1)

**Communication Protocols:**
- Protocols required for computers to properly communicate across the network
- Key protocol characteristics:
  - **Message format:** Specific format/structure required
  - **Message size:** Strict rules, vary by channel, long messages broken into smaller pieces
  - **Timing:** Speed of bit transmission, when host can send, total data amount per transmission
  - **Encoding:** Bits encoded into sounds, light waves, or electrical impulses (depends on media)
  - **Encapsulation:** Adding header with addressing info (source and destination)
  - **Message pattern:** Some require acknowledgment (request/response), others stream without delivery confirmation

**Communication Standards:**
- Standards ensure all devices implement same rules/protocols in same manner
- Enable different device types to communicate over the internet
- Development process: discussion, problem solving, testing
- **IETF:** Publishes and manages internet standards
- **RFC (Request for Comments):** Numbered documents tracking evolution of standards

**Network Communication Models:**
- Protocols can be illustrated as a protocol stack (layered hierarchy)
- Each higher-level protocol depends on services of lower levels
- Separation of functions enables each layer to operate independently

**TCP/IP protocol suite structure:**
- **Application:** Represents data to user, plus encoding and dialog control
- **Transport:** Supports communication between devices across diverse networks
- **Internet:** Determines best path through network
- **Network Access:** Hardware devices and media that make up the network

**OSI Reference Model:**
- Aid for clearer understanding of functions and processes
- Used for data network design, operation specifications, and troubleshooting
- Seven layers (Application, Presentation, Session, Transport, Network, Data Link, Physical)
- Each layer has specific functions and responsibilities

---

---

## Feb 19, 2026

## Module 6: Network Media

### Module objective
Describe common network media types and their characteristics.

---

### What is Network Media?

**Core concept:**
- Media provides the channel over which messages travel from source to destination
- Data is transmitted across a network on physical or wireless media

**Three primary media types in modern networks:**
1. Metal wires within cables (electrical impulses)
2. Glass or plastic fibers within cables (light pulses)
3. Wireless transmission (electromagnetic waves)

---

### Criteria for Choosing Network Media

**Four main selection factors:**

| Factor | Question to Ask |
|--------|----------------|
| **Distance** | What is the maximum distance the media can successfully carry a signal? |
| **Environment** | What is the environment in which the media will be installed? |
| **Speed/Capacity** | What is the amount of data and at what speed must it be transmitted? |
| **Cost** | What is the cost of the media and installation? |

---

### Three Common Network Cables

#### Twisted-Pair Cable

**Characteristics:**
- Most commonly encountered network cabling type
- Foundation of Ethernet technology
- Wires grouped in pairs and twisted together to reduce interference

**Wire identification:**
- Pairs are color-coded
- Each pair: one solid color wire + one striped wire 

**Common uses:**
- Ethernet connections in LANs
- Connecting devices to switches and routers
- Most local network deployments

---

#### Coaxial Cable

**Characteristics:**
- One of the earliest types of network cabling
- Single rigid copper core conducts the signal
- Layers: copper core → insulation → braided metal shielding → protective jacket

**Properties:**
- High-frequency transmission line
- Carries high-frequency or broadband signals

**Common uses:**
- Cable TV distribution
- Satellite communication systems
- Connecting satellite components

---

#### Fiber-Optic Cable

**Characteristics:**
- Glass or plastic fibers
- Diameter approximately the same as a human hair
- Uses light instead of electricity to carry data

**Key advantages:**
- Very high bandwidth (carries large amounts of data)
- High speeds over long distances
- Immune to electrical interference (uses light, not electricity)

**Common uses:**
- Backbone networks
- Large enterprise environments
- Large data centers
- Telephone company infrastructure
- Medical imaging and treatment
- Mechanical engineering inspection

---

### Media Comparison Summary

| Media Type | Signal Method | Interference Susceptibility | Distance | Bandwidth | Common Use |
|-----------|--------------|---------------------------|----------|-----------|------------|
| **Twisted-Pair** | Electrical impulses | Moderate (reduced by twisting) | Short to medium | Good | LANs, Ethernet |
| **Coaxial** | Electrical impulses | Low (shielded) | Medium | High | Cable TV, satellite |
| **Fiber-Optic** | Light pulses | None (uses light) | Very long | Very high | Backbones, data centers |
| **Wireless** | Electromagnetic waves | Moderate (interference possible) | Varies | Varies | Wi-Fi, cellular |

---

### Key Takeaways

**Media selection is context-dependent:**
- No single "best" media for all situations
- Choice depends on distance, environment, speed requirements, and budget

**Twisted-pair dominates LANs:**
- Ethernet uses twisted-pair extensively
- Most common cable type encountered in local networks

**Fiber for high-performance:**
- Best for long distances and high bandwidth
- Immune to electrical interference
- Used in enterprise backbones and data centers

**Coaxial for specialized use:**
- Cable TV and satellite infrastructure
- High-frequency signal transmission

---

## Feb 21, 2026

## Module 7: The Access Layer

### Module objective
Explain how communication occurs on Ethernet networks.

**Topics covered:**
- Encapsulation and the Ethernet Frame
- The Access Layer

---

### Encapsulation basics

**Human communication analogy:**
- Writing a letter: Use accepted format to ensure delivery and understanding
- Letter inside envelope = encapsulation
- Recipient removes letter from envelope = de-encapsulation

**Network communication:**
- Computer messages follow specific format rules
- Messages must be correctly formatted to be delivered and processed
- Incorrectly formatted messages are not successfully delivered

---

### What is a frame?

**Definition:**
- Each computer message is encapsulated in a specific format called a **frame**
- Frame acts like an envelope

**Frame contents:**
- Address of intended destination
- Address of source host
- Format and contents determined by:
  - Type of message being sent
  - Channel over which it is communicated

---

### Encapsulation process

**Placing one message format inside another:**
- Letter analogy: Letter goes inside envelope
- Network: Data encapsulated in frame for network transmission

**De-encapsulation:**
- Recipient reverses the process
- Removes the original message from the frame/envelope

**Example with IP:**
- Internet Protocol (IP) functions like the envelope
- IPv6 packet fields identify:
  - Source of packet
  - Destination of packet
- IP responsible for sending message from source to destination over one or more networks

---

### Ethernet frame format

**What Ethernet protocol standards define:**
- Frame format
- Frame size
- Timing
- Encoding

**Ethernet frame fields:**
- **Preamble:** Sequencing and timing
- **Start of frame delimiter**
- **Destination MAC address:** Where frame is going
- **Source MAC address:** Where frame came from
- **Length and type of frame**
- **Frame check sequence:** Detects transmission errors

---

### MAC addresses

**What they are:**
- Media Access Control (MAC) address
- Unique address permanently embedded on Ethernet NIC
- Every Ethernet NIC has unique MAC address

**Usage:**
- Source and destination MAC addresses are fields in Ethernet frame
- Ethernet is technology commonly used in local area networks
- Devices access Ethernet LAN using Ethernet Network Interface Card (NIC)

---

### The access layer

**What it is:**
- Part of network where people gain access to:
  - Other hosts
  - Shared files
  - Printers
- Provides first line of networking devices connecting hosts to wired Ethernet network

**Connection method:**
- Each host can connect directly to access layer networking device
- Connection uses Ethernet cable

---

### Ethernet hubs (obsolete)

**Characteristics:**
- Contain multiple ports to connect hosts to network
- Only one message can be sent through hub at a time

**Problem: Collisions**
- Two or more messages sent simultaneously cause collision
- Excessive retransmissions clog network
- Slow down network traffic

**Current status:**
- Now considered obsolete
- Replaced by Ethernet switches

---

### Ethernet switches

**What they are:**
- Device used at Layer 2 (Data Link layer)
- Modern replacement for hubs

**How they work:**
1. Host sends message to another host on same switched network
2. Switch accepts and decodes frames
3. Switch reads MAC address portion of message
4. Switch checks MAC address table
5. If destination found, switch builds temporary circuit between source and destination ports

---

### MAC address table

**What it contains:**
- List of all active ports
- Host MAC addresses attached to each port

**How it's built:**
- Switch examines source MAC address of each frame
- When new host sends message or responds to flooded message:
  - Switch immediately learns its MAC address
  - Switch records port to which it is connected
- Table is dynamically updated each time new source MAC address is read

---

### Switch advantages over hubs

**Collision elimination:**
- Switches allow sending and receiving frames simultaneously
- Can use same Ethernet cable for both directions at once
- Improves network performance by eliminating collisions

**Selective forwarding:**
- Switch only forwards frame to destination port (if known)
- Not broadcast to all ports like hub
- Reduces unnecessary traffic

---

### Module 7 summary

**Key concepts covered:**

**Encapsulation:**
- Process of placing one message format inside another
- De-encapsulation reverses the process
- Computer messages must follow specific format rules
- Frame = envelope for network message

**Ethernet frame format:**
- Standards define: frame format, size, timing, encoding
- Key fields: preamble, start delimiter, source/destination MAC addresses, length/type, frame check sequence

**MAC addresses:**
- Unique identifier permanently embedded on Ethernet NIC
- Used for source and destination addressing in frames

**Access layer function:**
- Where people gain access to network resources
- Provides first line of networking devices
- Connects hosts to wired Ethernet network

**Ethernet hubs (obsolete):**
- Multiple ports for host connections
- Only one message at a time
- Collisions slow network
- Replaced by switches

**Ethernet switches:**
- Layer 2 devices
- Use MAC address table to forward frames
- Build temporary circuits between source and destination
- Allow simultaneous send/receive on same cable
- Eliminate collisions
- Dynamically learn MAC addresses from traffic

---

---

## Feb 24, 2026

## Module 8: The Internet Protocol

### Module objective
Explain the features of an IP address and how IPv4 addresses identify hosts on networks.

**Topics covered:**
- Purpose of an IPv4 Address
- The IPv4 Address Structure

---

### Purpose of an IPv4 Address

**Core concept:**
- IPv4 address is a logical network address that identifies a particular host
- Required for participation on the internet and almost all LANs

**Address requirements:**
- Must be properly configured and unique within the LAN (local communication)
- Must be properly configured and unique in the world (remote communication)

**Where IPv4 addresses are assigned:**
- Network interface connection (usually a NIC)
- Each network interface gets its own IPv4 address
- Examples: workstations, servers, network printers, IP phones
- Routers have IPv4 addresses on each interface

**Critical for packet delivery:**
- Every packet has source and destination IPv4 addresses
- Networking devices use these addresses to route packets to destination
- Return path uses source address to send replies back

---

### IPv4 Address Format

**Binary representation:**
- IPv4 addresses are 32 bits in length
- Example: `11010001101001011100100000000001`

**Octets (grouped bits):**
- 32 bits grouped into four 8-bit bytes called octets
- Example: `11010001.10100101.11001000.00000001`

**Dotted-decimal notation:**
- Each octet converted to decimal value (0-255)
- Separated by decimal points (periods)
- Example: `209.165.200.1`

**Why dotted-decimal:**
- Much easier for humans to read and configure
- Standard format for representing IPv4 addresses
- Binary still used by networking devices internally

---

### IPv4 Address Structure (Hierarchical)

**Two-part structure:**
- **Network portion:** Identifies which network the host is on
- **Host portion:** Identifies the specific host on that network

**Subnet mask role:**
- Used to identify the network portion of an IPv4 address
- Determines which bits represent the network vs host

**Example breakdown:**
- **IPv4 address:** `192.168.5.11`
- **Subnet mask:** `255.255.255.0`
- **Network portion:** `192.168.5` (first three octets)
- **Host portion:** `11` (last octet)

---

### Hierarchical Addressing Benefits

**Router efficiency:**
- Routers only need to know how to reach each network
- Don't need to track location of every individual host
- Network portion provides routing direction

**Analogy: Telephone system**
- Country code, area code, exchange = network address
- Remaining digits = local phone number (host)

**Analogy: Postal system**
- Postal code and city = network portion
- Street address = host portion
- Post office routes to city (network), then to specific address (host)

---

### Multiple Logical Networks on One Physical Network

**Key concept:**
- Multiple logical networks can exist on one physical network
- Determined by different network portions in IPv4 addresses

**Example scenario:**
- **Physical network:** One Ethernet switch connecting 6 hosts
- **Logical network 1:** Three hosts with addresses `192.168.18.x` (network: `192.168.18`)
- **Logical network 2:** Three hosts with addresses `192.168.5.x` (network: `192.168.5`)

**Communication rules:**
- Hosts with same network portion can communicate directly
- Hosts with different network portions require routing to communicate
- Result: One physical network, two logical IPv4 networks

---

### IPv4 Address Components Summary

| Component | Description | Example |
|-----------|-------------|---------|
| **Full IPv4 Address** | 32-bit logical address in dotted-decimal | `192.168.5.11` |
| **Network Portion** | Identifies which network | `192.168.5` |
| **Host Portion** | Identifies specific host on network | `11` |
| **Subnet Mask** | Indicates network/host division | `255.255.255.0` |

---

### Key Takeaways

**IPv4 address fundamentals:**
- 32-bit address divided into 4 octets (8 bits each)
- Represented in dotted-decimal notation for readability
- Must be unique within LAN and globally unique for internet communication

**Hierarchical structure:**
- Two parts: network portion + host portion
- Subnet mask determines the division
- Network portion enables efficient routing

**Network vs host:**
- Network portion = which network (like postal code/city)
- Host portion = specific device (like street address)
- Routers route based on network portion only

**Logical vs physical networks:**
- Physical network = actual cables/switches connecting devices
- Logical network = devices sharing same network portion in addresses
- Multiple logical networks can exist on one physical network

**Communication rules:**
- Same network portion → direct communication
- Different network portion → requires router

---

---

## Feb 26, 2026

## Module 9: IPv4 and Network Segmentation

### Module objective
Explain how IPv4 addresses are used in network communication and segmentation.

**Topics covered:**
- IPv4 Unicast, Broadcast, and Multicast
- Types of IPv4 Addresses
- Network Segmentation

---

### IPv4 Transmission Types

**Three ways to transmit IPv4 packets:**

| Type | Description | Example Use |
|------|-------------|-------------|
| **Unicast** | One-to-one communication | Standard client-server communication |
| **Broadcast** | One-to-all communication | DHCP discovery, ARP requests |
| **Multicast** | One-to-many (selected group) | Routing protocol updates, streaming |

---

### Unicast Transmission

**Definition:**
- One device sending a message to one other device
- Standard form of network communication

**Characteristics:**
- Destination IP is a unicast address (single recipient)
- Source IP is always unicast (packet originates from single source)
- Most network traffic is unicast

**Unicast address range:**
- `1.1.1.1` to `223.255.255.255`
- However, many addresses in this range are reserved for special purposes

**Note:** All communication in this course is unicast unless otherwise specified.

---

### Broadcast Transmission

**Definition:**
- Device sending a message to all devices on the network
- One-to-all communication

**Characteristics:**
- Destination IP has all 1s in the host portion
- All devices in the broadcast domain must process the packet

**Two types of broadcast:**

**1) Limited broadcast:**
- Address: `255.255.255.255`
- Sent to all devices on local network
- Not forwarded by routers

**2) Directed broadcast:**
- Sent to all hosts on a specific network
- Example: `172.16.4.255` (all hosts on `172.16.4.0/24`)
- Router forwards to specific network

**Important:** Routers do not forward broadcasts by default.

**Performance impact:**
- Broadcasts consume network resources
- Every host must process broadcast packets
- Excessive broadcasts degrade network performance
- Solution: Segment networks to limit broadcast domains

**Note:** IPv6 does not use broadcasts (uses multicast instead).

---

### Multicast Transmission

**Definition:**
- Host sends single packet to selected group of hosts
- One-to-many (subscribers only)

**Characteristics:**
- Destination IP is a multicast address
- Only subscribed hosts process the packet
- More efficient than sending multiple unicast packets

**Multicast address range:**
- IPv4 reserved: `224.0.0.0` to `239.255.255.255`

**How it works:**
- Hosts subscribe to multicast group
- Each group has unique multicast IP address
- Subscribers process packets to that multicast address AND their unicast address

**Example use case:**
- OSPF routing protocol uses `224.0.0.5`
- Only OSPF-enabled routers process these packets
- Other devices ignore them

---

### Public vs Private IPv4 Addresses

**Public addresses:**
- Globally routable on the internet
- Must be unique worldwide
- Assigned by Regional Internet Registries (RIRs)
- Used for internet-facing resources

**Private addresses (RFC 1918):**
- Used internally within organizations
- NOT globally routable
- Can be reused by different organizations
- Must be translated to public address to reach internet

**RFC 1918 Private Address Ranges:**

| Network | Address Range | Typical Use |
|---------|--------------|-------------|
| `10.0.0.0/8` | `10.0.0.0` - `10.255.255.255` | Large enterprises |
| `172.16.0.0/12` | `172.16.0.0` - `172.31.255.255` | Medium organizations |
| `192.168.0.0/16` | `192.168.0.0` - `192.168.255.255` | Home networks, small businesses |

**Note:** Private addresses introduced in mid-1990s due to IPv4 address depletion. Long-term solution is IPv6.

---

### Network Address Translation (NAT)

**Purpose:**
- Translate private IPv4 addresses to public IPv4 addresses
- Enables private networks to access the internet

**How it works:**
1. Internal device (private IP) sends packet to internet
2. Router (usually at network edge) receives packet
3. Router translates source IP from private to public
4. Router forwards packet to ISP with public source IP
5. Return traffic translated back to private IP

**Key point:** Private addresses cannot be routed on the internet without NAT.

---

### Special Use IPv4 Addresses

**Loopback addresses:**
- Range: `127.0.0.0/8` (`127.0.0.1` to `127.255.255.254`)
- Commonly: `127.0.0.1`
- Purpose: Host directs traffic to itself
- Use case: Test local IP configuration

**Example:**
```bash
ping 127.0.0.1
# Tests if IP stack is working on local device
```

**Link-local addresses:**
- Range: `169.254.0.0/16` (`169.254.0.1` to `169.254.255.254`)
- Also called: APIPA (Automatic Private IP Addressing)
- Purpose: Self-assigned when DHCP unavailable
- Use case: Windows client cannot reach DHCP server
- Limitation: Can only communicate within local subnet

---

### Legacy Classful Addressing

**Historical context:**
- Introduced in 1981 (RFC 790)
- Divided unicast addresses into classes A, B, C
- Deprecated in mid-1990s due to inefficiency

**Three main classes:**

| Class | Address Range | Network Size | Host Addresses |
|-------|--------------|--------------|----------------|
| **A** | `0.0.0.0/8` to `127.0.0.0/8` | Very large | 16+ million per network |
| **B** | `128.0.0.0/16` to `191.255.0.0/16` | Moderate to large | ~65,000 per network |
| **C** | `192.0.0.0/24` to `223.255.255.0/24` | Small | 254 per network |

**Additional classes:**
- **Class D:** `224.0.0.0` to `239.0.0.0` (multicast)
- **Class E:** `240.0.0.0` to `255.0.0.0` (experimental)

**Why it was deprecated:**
- Class A and B had too many addresses (wasteful)
- Class C had too few addresses (limiting)
- Class A networks were 50% of IPv4 space but mostly unused
- Replaced by classless addressing (CIDR)

**Modern approach:**
- Classless addressing (used today)
- Addresses allocated based on actual need
- More efficient use of IPv4 address space

---

### IPv4 Address Assignment Hierarchy

**IANA (Internet Assigned Numbers Authority):**
- Top-level authority for IP address management
- Allocates blocks to Regional Internet Registries (RIRs)

**Five Regional Internet Registries:**
- **ARIN:** North America
- **RIPE NCC:** Europe, Middle East, parts of Asia
- **APNIC:** Asia Pacific
- **LACNIC:** Latin America and Caribbean
- **AfriNIC:** Africa

**Address allocation flow:**
```
IANA → RIRs → ISPs → Organizations/Smaller ISPs
```

**Organizations can also:**
- Get addresses directly from RIR (subject to RIR policies)

---

### Broadcast Domains and Routers

**Broadcast domain:**
- Collection of devices that receive broadcast packets
- Switches propagate broadcasts to all ports (except source port)

**Router behavior:**
- Routers do NOT forward broadcasts
- Each router interface connects to separate broadcast domain
- Broadcast received on one interface is NOT forwarded to other interfaces

**Why this matters:**
- Routers segment broadcast domains
- Limits broadcast traffic to specific networks
- Improves network performance

---

### Problems with Large Broadcast Domains

**Issues with large networks:**
- Many hosts can generate excessive broadcasts
- Every device must process every broadcast
- Results in slow network operations
- Results in slow device operations

**Example problem:**
- LAN with 400 users (`172.16.0.0/16`)
- All 400 users in single broadcast domain
- High broadcast traffic affects all users

**Solution: Subnetting**
- Divide large network into smaller subnets
- Creates smaller broadcast domains
- Reduces broadcast traffic impact

**Example solution:**
- Original: `172.16.0.0/16` (400 users)
- Subnet 1: `172.16.0.0/24` (200 users)
- Subnet 2: `172.16.1.0/24` (200 users)
- Broadcast in Subnet 1 does NOT reach Subnet 2

**Key concept:**
- Longer prefix length = smaller network
- `/16` changed to `/24` = using host bits for subnets

---

### Benefits of Network Segmentation

**Subnetting advantages:**
1. **Reduces network traffic:** Limits broadcast propagation
2. **Improves performance:** Fewer broadcasts per segment
3. **Enables security policies:** Control which subnets can communicate
4. **Limits broadcast impact:** Misconfigurations affect smaller group

**Common subnetting strategies:**

**By location:**
- Building 1: `192.168.1.0/24`
- Building 2: `192.168.2.0/24`
- Building 3: `192.168.3.0/24`

**By group/function:**
- Sales: `10.1.0.0/24`
- Engineering: `10.2.0.0/24`
- HR: `10.3.0.0/24`

**By device type:**
- Workstations: `172.16.1.0/24`
- Servers: `172.16.2.0/24`
- Printers: `172.16.3.0/24`

---

### Key Networking Protocols Using Broadcasts

**ARP (Address Resolution Protocol):**
- Discovers MAC address from known IP address
- Sends Layer 2 broadcast on local network
- Essential for Ethernet communication

**DHCP (Dynamic Host Configuration Protocol):**
- Assigns IP configuration to hosts
- Client sends broadcast to find DHCP server
- Server responds with IP address assignment

**Both protocols:**
- Rely on broadcasts for discovery
- Confined to broadcast domain (not forwarded by routers)

---

### Key Takeaways

**IPv4 transmission types:**
- Unicast = one-to-one (most traffic)
- Broadcast = one-to-all (DHCP, ARP)
- Multicast = one-to-many subscribers (routing protocols)

**Address categories:**
- Public = globally routable, must be unique
- Private = internal use, not routable (10.x, 172.16.x, 192.168.x)
- Special = loopback (127.x), link-local (169.254.x)

**NAT necessity:**
- Private addresses cannot reach internet without translation
- NAT translates private → public at network edge

**Broadcast domains:**
- Switches propagate broadcasts (all ports)
- Routers block broadcasts (segment domains)
- Each router interface = separate broadcast domain

**Network segmentation benefits:**
- Reduces broadcast traffic
- Improves performance
- Enables security policies
- Limits impact of network issues

**Subnetting basics:**
- Uses host bits to create smaller networks
- Longer prefix = smaller network
- Example: `/16` → `/24` (borrowed 8 host bits)

**Legacy vs modern:**
- Classful addressing (A, B, C) = deprecated, wasteful
- Classless addressing (CIDR) = modern, efficient

---

---

## Feb 28, 2026

## Module 10: IPv6 Addressing Formats and Rules

### Module objective
Explain the features of IPv6 addressing and the need for IPv6 transition.

**Topics covered:**
- IPv4 Issues
- IPv6 Addressing

---

### The Need for IPv6

**Primary motivation:**
- IPv4 address space depletion
- 4.3 billion theoretical maximum addresses insufficient

**IPv6 solution:**
- 128-bit address space
- 340 undecillion (340 followed by 36 zeros) possible addresses
- Effectively unlimited addresses for foreseeable future

---

### Regional Internet Registry (RIR) IPv4 Exhaustion

**Status as of course date:**
- 4 out of 5 RIRs have exhausted IPv4 address pools
- Only limited IPv4 addresses available for allocation

**Temporary mitigation (IPv4):**
- Private addressing (RFC 1918)
- Network Address Translation (NAT)
- Extended IPv4 lifespan but created limitations

---

### Limitations of NAT

**Problems with NAT:**
- Creates latency (address translation overhead)
- Breaks end-to-end connectivity model
- Severely limits peer-to-peer communications
- Complicates many applications
- Not a long-term solution

**Why NAT isn't enough:**
- Increasing mobile device adoption
- Internet of Things (IoT) growth
- Emerging applications requiring direct connectivity

---

### IPv6 Enhancements Beyond Address Space

**Beyond just more addresses:**
- Simplified header format (more efficient routing)
- Built-in IPsec support
- No broadcast (uses multicast instead)
- Improved address autoconfiguration

**ICMPv6 improvements:**
- Address resolution (replaces ARP)
- Address autoconfiguration (SLAAC)
- Neighbor discovery
- Enhanced diagnostics

---

### Internet of Things (IoT) Impact

**Device explosion:**
- Traditional: Computers, tablets, smartphones
- Emerging: Automobiles, biomedical devices, household appliances, sensors
- Future: Natural ecosystem monitoring, wearables, industrial sensors

**IPv6 necessity:**
- Each IoT device needs unique address
- Billions of devices coming online
- IPv4 address space cannot accommodate growth

---

### IPv6 Adoption Status

**Mobile providers:**
- Leading IPv6 adoption
- Top 2 US mobile providers: >90% traffic over IPv6
- Mobile-first approach due to address needs

**Major content providers:**
- YouTube, Facebook, Netflix: IPv6 enabled
- Microsoft, Facebook, LinkedIn: Transitioning to IPv6-only internally

**ISPs:**
- Comcast (2018): >65% IPv6 deployment
- British Sky Broadcasting: >86% deployment
- Trend: Increasing IPv6 adoption worldwide

---

### IPv4 and IPv6 Coexistence

**Reality:**
- No specific cutover date
- Both protocols will coexist for years
- Gradual transition, not overnight switch

**Three migration techniques:**

---

**1) Dual Stack (Preferred):**
- Devices run both IPv4 and IPv6 simultaneously
- Also called "native IPv6"
- Network has IPv6 connection to ISP
- Can access both IPv4 and IPv6 content
- Most flexible approach

---

**2) Tunneling:**
- Transports IPv6 packets over IPv4 network
- IPv6 packet encapsulated inside IPv4 packet
- Used when IPv6 not available end-to-end
- Temporary solution during transition

---

**3) Translation (NAT64):**
- Allows IPv6 devices to communicate with IPv4 devices
- Translates between IPv6 and IPv4 packets
- Similar concept to IPv4 NAT
- Used when IPv6-only needs to reach IPv4-only

**Goal:** Native IPv6 end-to-end communication. Tunneling and translation are temporary transition mechanisms.

---

### IPv6 Address Representation

**Address length:**
- 128 bits (vs IPv4's 32 bits)
- Written as hexadecimal string

**Hexadecimal number system:**
- Base 16 (0-9, A-F)
- Digits: 0 1 2 3 4 5 6 7 8 9 A B C D E F
- Case insensitive (A = a, F = f)

---

### IPv6 Address Format

**Structure:**
- 32 hexadecimal digits total
- Grouped into 8 segments (hextets)
- Each hextet = 16 bits = 4 hex digits
- Format: `x:x:x:x:x:x:x:x`

**Terminology:**
- **Octet:** 8 bits (IPv4 term)
- **Hextet:** 16 bits (IPv6 term, unofficial but common)

**Example (preferred format):**
```
2001:0db8:0000:1111:0000:0000:0000:0200
```

**Characteristics:**
- Colons separate hextets
- Leading zeros present in each hextet
- All 32 hex digits shown

---

### Rule 1: Omit Leading Zeros

**Purpose:** Reduce address notation length

**Rule:**
- Can omit leading zeros in each hextet
- CANNOT omit trailing zeros (would be ambiguous)

**Examples:**

| Preferred Format | Leading Zeros Omitted |
|-----------------|----------------------|
| `2001:0db8:0000:1111:0000:0000:0000:0200` | `2001:db8:0:1111:0:0:0:200` |
| `2001:0db8:000a:0001:c012:90ff:fe90:0001` | `2001:db8:a:1:c012:90ff:fe90:1` |
| `fe80:0000:0000:0000:0123:4567:89ab:cdef` | `fe80:0:0:0:123:4567:89ab:cdef` |

**Why trailing zeros can't be omitted:**
- `abc` could be `0abc` or `abc0` (different values)
- Omitting trailing zeros creates ambiguity

---

### Rule 2: Double Colon Compression

**Purpose:** Further reduce notation for consecutive zero hextets

**Rule:**
- Double colon `::` replaces one contiguous string of all-zero hextets
- Can only be used ONCE per address
- Best practice: Use on longest string of zeros
- If equal length strings: Use on first occurrence

**Example:**
```
2001:db8:0:1111:0:0:0:200  →  2001:db8:0:1111::200
```
The `::` replaces three consecutive zero hextets `(0:0:0)`

---

**Why only once per address:**

**Incorrect usage:**
```
2001:db8::abcd::1234  ❌ WRONG
```

**Possible expansions (ambiguous):**
```
2001:db8::abcd:0000:0000:1234
2001:db8::abcd:0000:0000:0000:1234
2001:db8:0000:abcd::1234
2001:db8:0000:0000:abcd::1234
```

**Cannot determine which is intended!**

---

### IPv6 Address Compression Examples

**Example 1:**
```
Preferred:   2001:0db8:0000:1111:0000:0000:0000:0200
No leading:  2001:db8:0:1111:0:0:0:200
Compressed:  2001:db8:0:1111::200
```

**Example 2:**
```
Preferred:   2001:0db8:aaaa:0001:0000:0000:0000:0000
No leading:  2001:db8:aaaa:1:0:0:0:0
Compressed:  2001:db8:aaaa:1::
```

**Example 3:**
```
Preferred:   fe80:0000:0000:0000:0123:4567:89ab:cdef
No leading:  fe80:0:0:0:123:4567:89ab:cdef
Compressed:  fe80::123:4567:89ab:cdef
```

**Example 4 (special addresses):**
```
Loopback:
Preferred:   0000:0000:0000:0000:0000:0000:0000:0001
Compressed:  ::1

All zeros:
Preferred:   0000:0000:0000:0000:0000:0000:0000:0000
Compressed:  ::
```

---

### Address Compression Best Practices

**Multiple zero strings:**
- Compress the longest string
- If equal length, compress the first

**Example with multiple zero strings:**
```
2001:db8:0:0:ab00:0:0:0

Option A: 2001:db8::ab00:0:0:0  (compress first 0:0)
Option B: 2001:db8:0:0:ab00::   (compress last 0:0:0)

Best practice: Option B (longer string compressed)
```

---

### IPv6 Address Format Summary

**Three representation formats:**

| Format | Description | Example |
|--------|-------------|---------|
| **Preferred** | All 32 hex digits, with leading zeros | `2001:0db8:0000:1111:0000:0000:0000:0200` |
| **No leading zeros** | Leading zeros omitted | `2001:db8:0:1111:0:0:0:200` |
| **Compressed** | Leading zeros omitted + double colon | `2001:db8:0:1111::200` |

**All three represent the same address.**

---

### Key Takeaways

**Why IPv6:**
- IPv4 address exhaustion (4.3 billion insufficient)
- IoT device explosion requires more addresses
- NAT limitations hinder many applications
- 340 undecillion IPv6 addresses effectively unlimited

**IPv6 advantages:**
- Massive address space
- No need for NAT
- Improved routing efficiency
- Built-in security features
- Better support for mobile devices

**Coexistence strategy:**
- Dual stack preferred (run both protocols)
- Tunneling for transitional networks
- Translation for IPv6-only to IPv4-only
- Goal: Native IPv6 end-to-end

**IPv6 address format:**
- 128 bits (32 hex digits)
- 8 hextets of 16 bits each
- Separated by colons
- Hexadecimal (0-9, A-F)

**Compression rules:**
- Rule 1: Omit leading zeros (not trailing)
- Rule 2: Use `::` for consecutive zeros (once per address)
- Compressed format most common in practice

**Adoption reality:**
- Coexistence will last years
- Mobile providers leading adoption
- Major content providers IPv6-enabled
- Gradual transition, not immediate switch

---

## Mar 3, 2026

## Module 11: Dynamic Addressing with DHCP

### Module objective
Configure a DHCP server and understand dynamic address assignment.

**Topics covered:**
- Static and Dynamic Addressing
- DHCPv4 Configuration

---

### Static IPv4 Address Assignment

**Definition:**
- Network administrator manually configures network information on each host

**Minimum required configuration:**
- IP address - Identifies host on network
- Subnet mask - Identifies network host is connected to
- Default gateway - Router used to access internet or remote networks

---

### Advantages of Static Addressing

**Benefits:**
- Useful for servers, printers, network devices
- Provides consistent addressing for resources
- Increased control of network resources
- Predictable addressing for DNS, documentation

**Use cases:**
- Servers (web, database, file servers)
- Network printers
- Network infrastructure devices
- Devices that need consistent addressing

---

### Disadvantages of Static Addressing

**Problems:**
- Time-consuming to configure each host manually
- Error-prone (typos, duplicate IPs, wrong subnet masks)
- Host only performs basic error checks
- Requires maintaining accurate address assignment list
- Addresses are permanent, not reused
- Doesn't scale well for large networks

**Administrative burden:**
- Manual configuration for new devices
- Manual updates when network changes
- Tracking which addresses are assigned

---

### Dynamic IPv4 Address Assignment

**Definition:**
- Automatic assignment of IP configuration using DHCP
- Dynamic Host Configuration Protocol

**What DHCP provides automatically:**
- IPv4 address
- Subnet mask
- Default gateway
- DNS server addresses
- Other configuration information

---

### Advantages of DHCP

**Benefits:**
1. **Reduces administrative burden:** No manual configuration needed
2. **Eliminates entry errors:** Automated assignment prevents typos
3. **Addresses are leased:** Temporary assignment, not permanent
4. **Efficient address reuse:** Released addresses return to pool
5. **Mobility support:** Users can move between networks easily

**Ideal for:**
- Large networks with many hosts
- Networks with frequent user changes
- Mobile users (laptops, tablets, phones)
- Guest networks

---

### DHCP Lease Concept

**How leasing works:**
- Address assigned for specific time period
- Host renews lease while active
- If host powered down or leaves network, address returns to pool
- Same address may be reassigned to different host

**Benefits:**
- Efficient use of limited address space
- Automatic reclamation of unused addresses
- Supports transient users (coffee shops, airports)

---

### DHCP Server Deployment Options

**Enterprise/Large networks:**
- Dedicated DHCP server (usually Windows/Linux server)
- Centralized management
- Supports multiple subnets via DHCP relay

**Home/Small networks:**
- DHCP server on ISP side (rare)
- DHCP server built into wireless router (common)

**Wireless router dual role:**
- Acts as DHCP **client** to ISP (receives public IP)
- Acts as DHCP **server** for local network (distributes private IPs)

---

### DHCP in Different Scenarios

**Public wireless hotspots:**
- Airport, coffee shop, hotel wireless
- Laptop DHCP client contacts local DHCP server
- Receives temporary IP address for session duration

**Home network:**
- Wireless router receives public IP from ISP (as DHCP client)
- Wireless router provides private IPs to home devices (as DHCP server)
- NAT translates between private and public addresses

**Corporate network:**
- Dedicated DHCP server on internal network
- May use multiple DHCP servers for redundancy
- DHCP relay agents forward requests between subnets

---

### DHCPv4 Operation (DORA Process)

**Four-step process:**

**1) DHCP Discover (Client → Broadcast):**
- Client sends broadcast looking for DHCP servers
- Destination IP: `255.255.255.255` (broadcast)
- Destination MAC: `FF-FF-FF-FF-FF-FF` (broadcast)
- All hosts receive, only DHCP servers respond

**2) DHCP Offer (Server → Client):**
- Server responds with available IP address offer
- Includes IP address, subnet mask, lease duration
- May include default gateway, DNS servers

**3) DHCP Request (Client → Broadcast):**
- Client broadcasts acceptance of offered address
- Requests permission to use the offered IP
- Broadcast allows other DHCP servers to know address is taken

**4) DHCP Acknowledgment (Server → Client):**
- Server confirms the address assignment
- Client can now use the IP configuration
- Lease timer begins

**Acronym:** DORA (Discover, Offer, Request, Acknowledgment)

---

### DHCP Server Configuration

**DHCP server components:**

**Address pool:**
- Range of IP addresses available for assignment
- Example: `192.168.1.100` to `192.168.1.200`

**Excluded addresses:**
- Addresses not available for dynamic assignment
- Reserved for static devices (servers, printers)

**Lease duration:**
- How long address is assigned before renewal required
- Typical: 24 hours to 7 days

**Default gateway:**
- Router IP address for hosts to reach other networks

**DNS servers:**
- DNS server addresses for name resolution

---

### Home Wireless Router DHCP Setup

**Default configuration:**
- Internal interface IP: `192.168.0.1` (typical)
- Subnet mask: `255.255.255.0`
- DHCP server enabled by default
- Address pool: Usually `192.168.0.100-200`

**Access router configuration:**
- Open web browser
- Navigate to router IP (e.g., `192.168.0.1`)
- Login with admin credentials
- Navigate to DHCP settings

**Common settings:**
- Enable/disable DHCP server
- Starting IP address
- Maximum number of users
- Lease time

---

### DHCP Address Pool Planning

**Considerations:**

**Network size:**
- Determine maximum number of concurrent hosts
- Plan for growth

**Address exclusions:**
- Reserve addresses for static devices
- Typically exclude lower addresses for servers/infrastructure

**Example planning:**
```
Network: 192.168.1.0/24
Total addresses: 254 usable

Reserved (static):
- 192.168.1.1: Router/gateway
- 192.168.1.2-10: Servers
- 192.168.1.11-20: Printers
- 192.168.1.21-30: Network devices

DHCP pool:
- 192.168.1.100-254: Dynamic assignment (155 addresses)
```

---

### Static vs Dynamic Comparison

| Aspect | Static Assignment | Dynamic Assignment (DHCP) |
|--------|------------------|--------------------------|
| **Configuration** | Manual on each host | Automatic via DHCP |
| **Error rate** | Higher (human error) | Lower (automated) |
| **Administrative effort** | High | Low |
| **Address reuse** | No (permanent) | Yes (leased) |
| **Best for** | Servers, printers, infrastructure | Workstations, mobile devices |
| **Scalability** | Poor for large networks | Excellent |
| **Mobility** | Requires reconfiguration | Seamless |

---

### DHCP Troubleshooting Basics

**Common issues:**

**Client gets no IP address:**
- DHCP server not running
- No DHCP server on network
- Client not receiving DHCP offers
- Firewall blocking DHCP traffic

**Client gets wrong network:**
- Multiple DHCP servers responding
- Rogue DHCP server on network
- Incorrect DHCP server configuration

**Client gets APIPA address (169.254.x.x):**
- No DHCP server available
- Client cannot reach DHCP server
- Windows automatic private IP addressing fallback

---

### Key Takeaways

**Static addressing:**
- Manual configuration required
- Time-consuming, error-prone
- Best for servers and infrastructure
- Requires address tracking

**Dynamic addressing (DHCP):**
- Automatic configuration
- Reduces administrative burden
- Virtually eliminates entry errors
- Addresses are leased, not permanent
- Efficient address reuse

**DHCP process:**
- Four-step: Discover, Offer, Request, Acknowledgment (DORA)
- Uses broadcast for discovery
- Client-server communication

**DHCP deployment:**
- Can be dedicated server or router feature
- Wireless routers commonly dual role (client and server)
- Configured with address pool and lease duration

**Best practices:**
- Use DHCP for workstations and mobile devices
- Use static addressing for servers and infrastructure
- Plan address pools with room for growth
- Document excluded (static) address ranges
- Monitor DHCP server for pool exhaustion

**DHCP advantages over static:**
- Scalability for large networks
- Reduced configuration errors
- Support for mobile users
- Efficient address utilization
- Lower administrative overhead

---

## Mar 8, 2026

## Module 12: Gateways to other networks

### Module objective
Explain how routers connect networks together.

**Topics covered:**
- Network Boundaries
- Network Address Translation

---

### Routers as gateways

**What a router provides:**
- Gateway for hosts on one network to communicate with hosts on different networks
- Each interface connected to separate network
- IPv4 address on interface identifies which local network is directly connected

**Default gateway concept:**
- Every host must use router as gateway to other networks
- Each host must know IPv4 address of router interface
- Address known as **default gateway address**

**Configuration methods:**
- Statically configured on host, OR
- Received dynamically via DHCP

---

### Wireless router as DHCP server

**Automatic default gateway distribution:**
- Wireless router configured as DHCP server
- Automatically sends correct interface IPv4 address to hosts
- Address provided as default gateway address

**Benefits:**
- All hosts can use address to forward messages to ISP
- Enables access to hosts on internet
- Wireless routers usually set as DHCP servers by default

**What DHCP provides to clients:**
- IPv4 address
- Subnet mask
- Default gateway (router's internal interface IP)

---

### Routers as boundaries between networks

**Internal (inside) network:**
- Local hosts attached to wireless router
- Connected via Ethernet cable or wirelessly
- DHCP server provides addresses to these hosts

**Default addressing:**
- Most DHCP servers assign **private addresses** to internal network
- Not internet-routable public addresses
- Ensures internal network not directly accessible from internet by default

**Router internal interface:**
- Usually first host address on network
- Example: Network `192.168.1.0/24` → Router interface `192.168.1.1`

**Internal host addressing:**
- Must be assigned addresses within same network as router
- Either statically configured OR via DHCP

---

### External (outside) network

**ISP side configuration:**
- Many ISPs use DHCP servers
- Provide IPv4 addresses to internet side of wireless router
- Network assigned to internet side = external/outside network

**Wireless router as DHCP client:**
- When connected to ISP, acts as DHCP client
- Receives correct external network IPv4 address for internet interface
- ISPs usually provide internet-routable address
- Enables hosts connected to router to access internet

**Router dual role:**
- DHCP server for internal network (private addresses)
- DHCP client to ISP (receives public address)

---

### Router as network boundary

**Boundary function:**
- Wireless router serves as boundary between:
  - Local internal network (private)
  - External internet (public)

**Address translation requirement:**
- Internal hosts use private addresses
- Internet requires public addresses
- Router performs translation between the two

---

### Network Address Translation (NAT)

**Purpose:**
- Convert private addresses to internet-routable addresses
- Allow multiple internal hosts to share single public address

**How NAT works:**

**Outbound traffic:**
- Private (local) source IPv4 address translated to public (global) address
- Router replaces private address with its own public address

**Inbound traffic:**
- Process reversed for incoming packets
- Public address translated back to appropriate private address

**Key principle:**
- Only packets destined for other networks need translation
- Packets must pass through gateway for translation to occur

---

### NAT operation details

**What router receives from ISP:**
- Public address for internet interface
- Allows sending and receiving packets on internet

**What router provides to local clients:**
- Private addresses for internal network
- Default gateway address (router's internal interface)

**Many-to-one translation:**
- Wireless router translates many internal IPv4 addresses
- All share same public address via NAT
- Enables multiple devices to access internet with single public IP

---

### Private vs public addressing summary

**Internal network (private):**
- Private IP addresses (RFC 1918)
  - `10.0.0.0/8`
  - `172.16.0.0/12`
  - `192.168.0.0/16`
- Not routable on internet
- Multiple organizations can use same private ranges
- Requires NAT to reach internet

**External network (public):**
- Public IP address from ISP
- Routable on internet
- Globally unique
- Limited availability (IPv4 exhaustion)

---

### Module 12 summary

**Key concepts covered:**

- **Default gateway:**
  - Router interface IP address
  - Known by every host on network
  - Configured statically or received via DHCP
  - Used to reach other networks

- **Router as boundary:**
  - Separates internal (private) network from external (public) internet
  - Internal hosts use private addresses
  - External interface uses public address from ISP

- **Router dual role:**
  - DHCP server for internal network
  - DHCP client to ISP
  - Provides default gateway to internal hosts
  - Receives public address from ISP

- **Network Address Translation (NAT):**
  - Converts private addresses to public address
  - Enables multiple internal hosts to share single public IP
  - Private source address → public address (outbound)
  - Public destination address → private address (inbound)
  - Only translates packets destined for other networks
  - Translation occurs at gateway (router)

- **Address assignment:**
  - Internal hosts typically receive private addresses
  - Router receives public address from ISP
  - DHCP automates distribution of addresses and default gateway
  - Default gateway usually first address on internal network

---

## Mar 11, 2026

## Module 13: The ARP process

### Module objective
Explain how ARP enables communication on a network.

**Topics covered:**
- MAC & IP Address Roles
- Broadcast Containment

---

### Two primary addresses on Ethernet LAN

**Physical address (MAC address):**
- Used for NIC-to-NIC communications
- On same Ethernet network
- Never changes (burned into hardware)

**Logical address (IP address):**
- Used to send packet from source to destination
- Destination may be on same network or remote network
- Can change based on network location (DHCP)

---

### Destination on same network

**Scenario:** Host must send message but only knows destination IP address

**Problem:** Host needs MAC address of destination device

**Solution:** Address resolution

---

**Layer 2 (Ethernet frame) addressing:**
- Physical addresses deliver frame from one NIC to another
- On same network

**If destination IP on same network:**
- Destination MAC address = destination device's MAC

---

**Example: PC1 to PC2 (same network)**

**Network:** `192.168.10.0/24`

**Layer 2 Ethernet frame contains:**
- Destination MAC: `55-55-55` (PC2's MAC)
- Source MAC: `aa-aa-aa` (PC1's MAC)

**Layer 3 IP packet contains:**
- Source IPv4: `192.168.10.10` (PC1)
- Destination IPv4: `192.168.10.11` (PC2)

---

### Destination on remote network

**When destination IP on remote network:**
- Destination MAC = default gateway (router interface)

**Why:**
- Host cannot directly reach remote network
- Must send to router first
- Router forwards to destination

---

**Example: PC1 to PC2 (remote network)**

**PC1 network:** `192.168.10.0/24`
**PC2 network:** Different network (remote)

**Layer 2 frame (PC1 to Router):**
- Destination MAC: Router's interface MAC (default gateway)
- Source MAC: PC1's MAC

**Layer 3 packet (unchanged):**
- Source IPv4: PC1's IP
- Destination IPv4: PC2's IP (on remote network)

---

### Router frame processing

**Router receives frame:**
1. De-encapsulates Layer 2 information
2. Examines destination IPv4 address
3. Determines next-hop device
4. Encapsulates IPv4 packet in new frame for outgoing interface

**New frame characteristics:**
- New source MAC: Outgoing router interface MAC
- New destination MAC: Next-hop device MAC (or final destination if last hop)

---

**Example: R1 forwarding to R2**

**New Layer 2 frame:**
- Destination MAC: R2 G0/0/1 interface MAC
- Source MAC: R1 G0/0/1 interface MAC

**Layer 3 packet:** Unchanged (still PC1 to PC2)

---

### Frame encapsulation along path

**Key principle:**
- Along each link, IP packet encapsulated in frame
- Frame specific to data link technology (e.g., Ethernet)

**If next-hop is final destination:**
- Destination MAC = destination device's NIC MAC

---

### Address resolution protocols

**For IPv4 packets:**
- Address Resolution Protocol (ARP)
- Associates IP addresses with MAC addresses

**For IPv6 packets:**
- ICMPv6 Neighbor Discovery (ND)
- Similar function to ARP

---

### Broadcast domains

**What is a broadcast domain:**
- Local area network with one or more Ethernet switches
- All hosts within domain receive broadcast messages

**Broadcast behavior:**
- Host sends broadcast message
- Switches forward to every connected host on same local network
- Broadcast MAC address: `FF:FF:FF:FF:FF:FF` (all ones)

---

### Broadcast MAC address

**Structure:**
- 48-bit address
- All bits set to 1
- Hexadecimal representation: `FF:FF:FF:FF:FF:FF`
- Each F represents four 1s in binary

**Recognition:**
- All hosts recognize and accept broadcast MAC address
- Every host processes broadcast as if addressed directly to it

---

### Broadcast domain limitations

**Problem with too many hosts:**
- Broadcast traffic becomes excessive
- Network traffic increases as hosts added
- Limited by switch capabilities

**Performance improvement solution:**
- Divide network into multiple broadcast domains
- Use routers to separate broadcast domains
- Each broadcast domain = separate network segment

**Key principle:**
- Routers do NOT forward broadcasts
- Broadcasts contained within local network

---

### Access layer communication problem

**NIC acceptance criteria:**
- Only accepts frame if destination is:
  - Broadcast MAC address, OR
  - NIC's own MAC address

**Application behavior:**
- Most network applications use logical IP address
- To identify servers and clients

**The problem:**
- Sending host only has destination IP address
- Needs destination MAC address for frame
- How to determine correct MAC address?

---

### Address Resolution Protocol (ARP)

**Purpose:**
- Discover MAC address of host on same local network
- When only IPv4 address known

**ARP three-step process:**

**Step 1: ARP request (broadcast)**
- Sending host creates frame addressed to broadcast MAC
- Frame contains message with destination's IPv4 address
- Sent to all hosts on network

**Step 2: ARP reply (unicast)**
- Each host receives broadcast frame
- Compares IPv4 address in message with its own
- Host with matching IPv4 sends MAC address back to sender

**Step 3: ARP table storage**
- Sending host receives reply
- Stores MAC address and IPv4 address in ARP table
- Future frames sent directly without new ARP request

---

### ARP table benefits

**Efficiency:**
- Once MAC address in ARP table, no new ARP request needed
- Can send frames directly to destination

**Requirements:**
- ARP messages rely on broadcast frames
- All hosts in local IPv4 network must be in same broadcast domain

---

### IPv6 neighbor discovery

**IPv6 equivalent to ARP:**
- Neighbor Discovery (ND) protocol
- Uses ICMPv6
- Similar function to ARP for IPv4

---

### Module 13 summary

**Key concepts covered:**

- **Two address types:**
  - MAC address (physical, Layer 2) - NIC-to-NIC on same network
  - IP address (logical, Layer 3) - Source to destination (local or remote)

- **Same network communication:**
  - Destination MAC = destination device's MAC
  - Both Layer 2 and Layer 3 addresses point to same device

- **Remote network communication:**
  - Destination MAC = default gateway (router interface)
  - Layer 3 destination IP = final destination
  - Router de-encapsulates, examines IP, re-encapsulates with new MAC addresses

- **Frame encapsulation:**
  - Each link has own Layer 2 frame
  - IP packet remains same throughout path
  - MAC addresses change at each hop

- **Broadcast domain:**
  - Local network where broadcasts reach all hosts
  - Switches forward broadcasts to all connected hosts
  - Routers divide networks into multiple broadcast domains

- **Broadcast MAC address:**
  - `FF:FF:FF:FF:FF:FF` (all ones)
  - Recognized and accepted by all hosts
  - Used for ARP requests

- **ARP process:**
  1. Broadcast ARP request with target IP
  2. Target host replies with its MAC address
  3. Requesting host stores mapping in ARP table

- **Address resolution protocols:**
  - IPv4: Address Resolution Protocol (ARP)
  - IPv6: Neighbor Discovery (ND)

- **Broadcast limitations:**
  - Too many hosts = excessive broadcast traffic
  - Performance degradation
  - Solution: Divide into multiple broadcast domains with routers

- **Why MAC and IP both needed:**
  - IP address: Identifies where device is (network location)
  - MAC address: Identifies specific device on local network
  - IP gets data to correct network
  - MAC delivers to specific device on that network

---
