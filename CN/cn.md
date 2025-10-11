# Complete Computer Networking Course Notes

## Table of Contents
1. [Introduction to Networking](#introduction-to-networking)
2. [History of the Internet](#history-of-the-internet)
3. [Basic Concepts](#basic-concepts)
4. [Network Topologies](#network-topologies)
5. [OSI Model](#osi-model)
6. [TCP/IP Model](#tcpip-model)
7. [Application Layer](#application-layer)
8. [Transport Layer](#transport-layer)
9. [Network Layer](#network-layer)
10. [Data Link Layer](#data-link-layer)
11. [Physical Layer](#physical-layer)

---

## Introduction to Networking

### What is a Computer Network?
- **Simple Definition**: Computers connected together
- **Network**: Collection of connected computers
- **Internet**: Collection of computer networks on a global scale

### Full Form of COMPUTER
**C**ommonly **O**riented **M**achine **P**articularly **U**sed for **T**raining, **E**ducation, and **R**esearch

---

## History of the Internet

### The Cold War Era (1950s)

**Context**:
- Cold War between United States and Soviet Union
- Competition for scientific supremacy

**Sputnik Launch (1957)**:
- Soviet Union launched world's first satellite
- US government response: Created **ARPA** (Advanced Research Projects Agency)
- Goal: Keep USA at forefront of scientific discoveries

### ARPANET (First Network)

**Initial Locations** (4 sites):
1. MIT
2. Stanford
3. UCLA
4. University of Utah

**Key Features**:
- Used TCP (Transmission Control Protocol)
- Connected four major research institutions
- Computers were far apart, needed communication method
- Foundation of modern internet
- Over time, more computers and locations were added

### World Wide Web (WWW)

**Creator**: Tim Berners-Lee

**Problem It Solved**:
- Researchers wanted to share documents that reference other documents
- Needed automated way to link research papers
- Example: MIT sends document about apples with link to another apple document

**Purpose**:
- Store and access documents universally
- Allow hyperlinks between documents
- Universal access to information

**First Website**: info.cern.ch/hypertext/www/project.html

**Evolution**:
- Initially **no search engines** (navigation via hyperlinks only)
- You couldn't search - just followed links from page to page
- Could save indices but didn't scale
- **Yahoo** was first search engine
- Google came later
- Modern internet now has extensive search capabilities

---

## Basic Concepts

### Internet Infrastructure

**Physical Connections - Submarine Cables**:
- Internet is NOT in the cloud
- **Actually under the ocean** - wires laid underground and underwater
- Website to check: **submarinecablemap.com**

**India's Connections**:
- Chennai, Kochi (South India)
- Mumbai
- Connected to: Sri Lanka, Dubai, Oman, UAE, Singapore, Malaysia

**Cable Details**:
- Example: 28,000+ kilometers long (Japan → South Korea → China → Malaysia → India → UAE → Israel → Italy → UK)
- One single wire spread across continents
- Heavily guarded and buried on ocean floor
- Protected from marine life (sharks, fish cannot cut them)
- Owned by various companies (Google owns many)

**Why Cables Instead of Satellites?**:
- **Faster than satellite communication**
- Speed of light through fiber optic cables

### Connection Types

**Guided (Physical Path)**:
- Ethernet cables
- Optical fiber cables
- Coaxial cables
- Two computers connected with wire

**Unguided (No Single Path)**:
- Wi-Fi
- Bluetooth
- 3G, 4G, LTE, 5G
- Radio channels

### Network Types

#### LAN (Local Area Network)
**Coverage**: Small area (house, office)
- "Small" means area-based, not device-based
- Can connect 10,000 computers if in proximity
- Connection methods:
  - **Ethernet cables**
  - **Wi-Fi**
  - **Network adapters**
  - **Ethernet switches**

#### MAN (Metropolitan Area Network)
**Coverage**: Across a city
- Uses ISPs (Internet Service Providers)

#### WAN (Wide Area Network)
**Coverage**: Across countries
- Uses optical fiber cables
- Technologies:
  - **SONET** (Synchronous Optical Networking) - carries data using optical fiber
  - **Frame Relay** - connects LAN to WAN

**Important**: Internet is collection of all three - LANs connected via MANs connected via WANs

---

## Network Topologies

### 1. Bus Topology

**Structure**:
```
Computer -- Computer -- Computer
            |
        Backbone Cable
```

**How It Works**:
- One backbone cable
- All computers connected to backbone
- Only one device can send data at a time

**Limitations**:
- If backbone breaks, entire network fails
- Only one person can send data at a time

### 2. Ring Topology

**Structure**:
```
Computer A → Computer B → Computer C
     ↑                          ↓
Computer F ← Computer E ← Computer D
```

**How It Works**:
- Computers connected in circle
- Data passes through each device

**Example**: A to F goes through B and C

**Limitations**:
- If one cable breaks, network fails
- Unnecessary calls to intermediate devices
- A to F has to go through B, C, D, E

### 3. Star Topology

**Structure**:
```
    Computer
       |
Computer - HUB/SWITCH - Computer
       |
    Computer
```

**How It Works**:
- Central device (hub/switch)
- All computers connect to center
- A to B communicates via central device

**Limitations**:
- If central device fails, network fails

### 4. Tree Topology

**Structure**:
- Combination of bus and star
- Multiple star networks connected via bus
```
Star Network 1 -- Star Network 2 -- Star Network 3
(on backbone bus)
```

**Advantages**:
- Better fault tolerance
- More scalable than pure bus or star

### 5. Mesh Topology

**Structure**:
```
Every computer connected to every other computer
```

**Limitations**:
- **Expensive** (lots of cables)
- **Scalability issues**
- Adding new computer requires connecting to ALL existing computers
- Complex wiring

---

## IP Addresses and Ports

### IP Addresses

**Purpose**: Identify devices on network (like phone numbers in phonebook)

**Analogy**:
- Phone contacts: Name → Phone Number
- Internet: Domain Name → IP Address
- You call "John" → dials actual number
- You type "google.com" → connects to actual IP

**IPv4 Format**:
```
X.X.X.X (e.g., 192.168.1.1)
```
- Each X: 0-255
- 32-bit numbers (4 × 8 bits)
- Total possible: 2^32 ≈ 4.3 billion addresses

**Example IP Address**: 192.168.2.30
```
192.168     |  2.30
Network     |  Device
Address     |  Address
```

**Binary Representation**:
- IP: 5.6.9.4
- 5 in binary: 00000101 (8 bits)
- Each number is 8 bits
- Total: 32 bits

**Special Addresses - Loopback**:
- **127.0.0.0/8** - Reserved
- **127.0.0.1** (localhost): Loop back to own computer
- Used for testing
- Allows processes on your machine to communicate
- Device acts as both client and server
- Cannot go down if computer is running

### IPv6 (Future)

**Why Needed**:
- IPv4: 2^32 addresses may run out
- Smartphones, IoT devices increasing

**Characteristics**:
- **128 bits** (4 times larger than IPv4)
- 2^128 unique addresses
- 3.4 × 10^38 addresses

**Format**:
```
8 groups of hexadecimal numbers
a1b2:c3d4:e5f6:1234:5678:9abc:def0:1234
```
- Each group: 16-bit hexadecimal
- Total: 8 × 16 = 128 bits

**Representation Rules**:
- All zeros can be written as `::` or `0`
- Example: `1::9` means zeros in between
- Can use prefix notation: `/60` means first 60 bits fixed

**Challenges (Why not fully adopted)**:
- **Not backward compatible** with IPv4
- Devices configured for IPv4 cannot access IPv6 servers
- Requires infrastructure changes
- ISPs need to upgrade hardware

### Port Numbers

**Purpose**: Identify applications on a device

**Why Needed**:
- One device can run multiple internet applications
- IP identifies the computer
- Port identifies the application

**Characteristics**:
- **16-bit numbers**
- Range: 0 to 65,535 (2^16)

**Communication Example**:
```
You (WhatsApp)          Friend (WhatsApp)
IP: 192.168.1.1         IP: 203.45.67.89
Port: 5000              Port: 5001
```

### Port Categories

**1. Reserved Ports (0-1023)**:
- Cannot use for custom applications
- Examples:
  - **HTTP**: 80
  - **HTTPS**: 443
  - **SSH**: 22
  - **FTP**: 21
  - **SMTP**: 25
  - **DNS**: 53
  - **Telnet**: 23

**2. Registered Ports (1024-49151)**:
- For specific applications
- Examples:
  - **MongoDB**: 27017
  - **MySQL**: 3306
  - **SQL Server**: 1433

**3. Dynamic/Private Ports (49152-65535)**:
- Available for custom applications

**Ephemeral Ports**:
- Temporary ports for multiple instances
- Example: Multiple Chrome tabs open
- Application assigns random ports internally
- Freed after process ends
- Used on client side
- Server must have well-defined ports (clients need to know them)

### Global vs Local IP Addresses

**Scenario**: You have Wi-Fi router with 4 devices connected

**Setup**:
```
ISP → Modem/Router (Global IP: 203.45.67.89)
      ├─ Device 1 (Local IP: 192.168.1.2)
      ├─ Device 2 (Local IP: 192.168.1.3)
      └─ Device 3 (Local IP: 192.168.1.4)
```

**Global IP Address**:
- Assigned by ISP to modem/router
- Same for ALL devices (to outside world)
- Visible to external servers (Google sees this)

**Local IP Address**:
- Assigned by router to individual devices
- Unique within your network
- Not visible outside network

**DHCP (Dynamic Host Configuration Protocol)**:
- Automatically assigns local IP addresses
- Modem/router acts as DHCP server
- When device connects, requests IP
- Server has pool of IP addresses
- Assigns one to new device

**NAT (Network Address Translation)**:
- Maps between local and global IP
- Router knows which device made request
- Example:
  - Device 1 opens Google Chrome
  - Requests google.com
  - Router remembers: Device 1 made request
  - Google responds to global IP
  - Router forwards to Device 1
- Works at network/transport layer

**How Router Identifies Application**:
- Uses **Port numbers**
- Device 1 running: MongoDB, Chrome, Game
- All have same IP but different ports
- Router uses ports to route data correctly

---

## Internet Speed Measurement

### Units

**Mbps (Megabits per second)**:
- 1 Mbps = 1,000,000 bits/second (10^6)
- Mega = 6 zeros

**Gbps (Gigabits per second)**:
- 1 Gbps = 1,000,000,000 bits/second (10^9)

**Kbps (Kilobits per second)**:
- 1 Kbps = 1,000 bits/second (10^3)
- Very slow

**Important**: 
- Measured in **bits** not bytes
- 1 byte = 8 bits

### Upload vs Download

**Upload**: Sending data from your device
**Download**: Receiving data to your device

**Testing**: Use speedtest.net (Ookla) or similar services

---

## Protocols

### What is a Protocol?

**Definition**: Set of rules for how data is transferred

**Why Needed**:
- Different types of data need different handling
- **Email**: All data must arrive (use TCP)
- **Video call**: Some frames can drop (use UDP)
- **Secure file**: 100% data delivery required

**Examples of Different Protocols**:
```
Email      → Need all data → Use TCP
Video Call → Can lose frames → Use UDP
File Transfer → Need all data → Use TCP
```

**Who Creates Protocols**:
- **Internet Society** (internetsociety.org)
- **RFC** (Request for Comments) - submission process
- Usually submitted by professionals
- Anyone can submit with good idea
- Organizations like IETF (Internet Engineering Task Force)

---

## Network Devices

### Repeater
**Layer**: Physical Layer
**Function**:
- Regenerates signal (not amplifies)
- Prevents signal from becoming weak/corrupted
- Copies signal bit by bit to original strength
- **2-port device**

### Hub
**Layer**: Physical Layer
**Function**:
- **Multi-port repeater**
- Multiple ports instead of just 2
- Connects wires from different branches
- Used in star topology
**Limitation**:
- Cannot filter data
- Sends data packets to ALL connected devices
- No intelligence for best path

### Bridge
**Layer**: Data Link Layer
**Function**:
- Type of repeater with additional functionality
- Can filter content by reading MAC addresses
- Reads source and destination MAC addresses

### Switch
**Layer**: Data Link Layer
**Function**:
- **Multi-port bridge**
- Boosts performance and efficiency
- Can perform error checking before forwarding
- More efficient than hub

### Router
**Layer**: Network Layer
**Function**:
- Routes data packets based on IP addresses
- Connects LANs and WANs
- Has forwarding/routing tables
- Covered in detail in Network Layer

### Gateway
**Layer**: Multiple layers
**Function**:
- Passage to connect two networks
- Can work with different networking models/protocols
- Bridges different protocol networks

### Brouter
**Function**:
- Combination of Bridge + Router
- Offers functionality of both

---

## OSI Model (Open Systems Interconnection)

### Overview

**Purpose**: Standard way for computers to communicate

**Total Layers**: 7 (from top to bottom)

**Real-World Analogy - Amazon Order**:
```
1. You order online (Application Layer)
2. Amazon receives order (Presentation Layer)
3. Amazon prepares order (Session Layer)
4. Ships to local delivery (Transport Layer)
5. Transported to India (Network Layer)
6. Local delivery company (Data Link Layer)
7. Physical delivery (Physical Layer)
```

### Layer Breakdown

**Layers (Top to Bottom)**:
1. Application Layer
2. Presentation Layer
3. Session Layer
4. Transport Layer
5. Network Layer
6. Data Link Layer
7. Physical Layer

**Data Units by Layer**:
- Application/Presentation/Session: **Messages/Data**
- Transport: **Segments**
- Network: **Packets**
- Data Link: **Frames**
- Physical: **Bits**

### How Data Flows

**Sending (You → Friend)**:
```
Application Layer (WhatsApp message)
    ↓
Presentation Layer (convert to binary, encrypt, compress)
    ↓
Session Layer (establish connection)
    ↓
Transport Layer (divide into segments, add ports)
    ↓
Network Layer (add IP addresses, create packets)
    ↓
Data Link Layer (add MAC addresses, create frames)
    ↓
Physical Layer (convert to bits, electrical signals)
    ↓
INTERNET (through routers, ISPs)
    ↓
Friend's Physical Layer
    ↓
Friend's Data Link Layer
    ↓
Friend's Network Layer
    ↓
Friend's Transport Layer
    ↓
Friend's Session Layer
    ↓
Friend's Presentation Layer
    ↓
Friend's Application Layer (message received)
```

**Key Concept**: Each layer imagines it's talking directly to same layer on friend's device

---

## Layer 1: Application Layer

### Basics

**What It Is**:
- Layer where users interact
- Implemented in software
- Contains applications (browsers, WhatsApp, email)

**Location**: On your devices (end systems)

**What It Does**:
- Users interact with applications
- Send messages, files, emails
- Browse web

### Protocols

#### HTTP (Hypertext Transfer Protocol)
**Port**: 80

**What It Does**:
- Defines how web clients and servers communicate
- Tells how to request data
- Tells how server sends data back

**Request Types (Methods)**:
- **GET**: Request data from server
- **POST**: Send data to server (forms, registration)
- **PUT**: Put data at specific location
- **DELETE**: Delete data from server

**Status Codes**:
- **100-199**: Informational
- **200-299**: Success (200 = OK)
- **300-399**: Redirection
- **400-499**: Client error (404 = Not Found, 400 = Bad Request)
- **500-599**: Server error (500 = Internal Server Error)

**Characteristics**:
- **Stateless**: Doesn't store client information
- Uses **TCP** (ensures all data received)
- Application layer protocol
- Transport layer: TCP

**Seeing HTTP in Action**:
```
1. Open browser
2. Go to google.com
3. Right-click → Inspect → Network tab
4. Refresh page
5. See all GET/POST requests
6. See status codes, headers, data
```

**Request Headers Example**:
- Accept: text/html, application/json
- Accept-Encoding: gzip, deflate
- Accept-Language: en
- Connection: keep-alive
- Host: google.com

**Response Headers Example**:
- Content-Type: text/html
- Content-Length: 1234
- Date: [timestamp]
- Server: gws
- Set-Cookie: [cookie data]

#### HTTPS (HTTP Secure)
**Port**: 443
- Encrypted version of HTTP
- Uses SSL/TLS
- Secure connection

#### Cookies

**Problem HTTP Solves**:
- HTTP is stateless
- But we want to stay logged in
- Cart items should remain

**Solution**: Cookies

**What Is a Cookie**:
- Unique string stored in browser
- File on client's browser
- Set by server on first visit

**How It Works**:
```
1. First visit to amazon.com
   ← Server sets cookie
2. Browser stores cookie
3. Second visit to amazon.com
   → Browser sends cookie in request header
4. Server reads cookie from database
   ← Recognizes you, sends personalized data
```

**Cookie Data**:
- Stored in browser
- Sent with every request to that domain
- Server has matching data in database
- Cookie-user mapping

**Cookie Expiration**:
- Has expiration date (like real cookies!)
- Set-Cookie header includes expiry

**Third-Party Cookies**:
- Cookies set by domains you didn't visit
- Example: Visit site with embedded ad
- Ad domain sets cookie
- Can track you across sites
- Safari/browsers allow blocking these

#### FTP (File Transfer Protocol)
- File transfers
- Less used now (HTTP can transfer files)

#### SMTP (Simple Mail Transfer Protocol)
**Purpose**: Send emails
**Uses**: TCP (need all data)

**How Email Works (Send)**:
```
Sender (you)
    ↓
Your SMTP Server (gmail.com)
    ↓
[Connection established]
    ↓
Receiver's SMTP Server (yahoo.com)
    ↓
Receiver downloads when they log in
```

**Special Case**: Both on same service (e.g., both Gmail)
- Direct transfer within same server
- No connection between different servers needed

**Error Handling**:
- If receiver server offline
- Sender server keeps trying for few days
- Then gives up and notifies sender

**Commands to Check SMTP**:
```bash
nslookup -type=mx gmail.com
# Shows SMTP server information
# MX = Mail Exchange = SMTP servers
```

#### POP3 (Post Office Protocol v3)
**Port**: 110
**Purpose**: Receive/download emails

**How It Works**:
```
1. Client connects to POP server (using TCP)
2. Authorization (username/password)
3. Transaction (get all emails)
4. Update (close session, server updates)
```

**Options**:
- Download and delete from server
  - Emails only on this device
  - Not available on other devices
- Download and keep on server
  - Available on multiple devices

**Limitation**:
- Other folders (sent, drafts) not synced

#### IMAP (Internet Message Access Protocol)
**Purpose**: Receive emails (more advanced than POP3)

**Advantages over POP3**:
- View emails on multiple devices
- Emails kept on server forever
- All folders synced (sent, drafts, etc.)
- Local copies on various devices
- Delete on one device → deleted everywhere

**Example**:
- Delete email on iPhone
- Also deleted on Mac, iPad, etc.

#### SSH (Secure Shell)
**Port**: 22
**Purpose**: Remote terminal access
**Use Case**: Log into another computer's terminal
- Used heavily in cloud computing
- Access EC2 instances
- Secure encrypted connection

#### Telnet
**Port**: 23
**Purpose**: Terminal emulation
**How It Works**: `telnet hostname` connects to that host
**Problem**: Not encrypted/encoded (unlike SSH)
**Data**: Plain text (can be intercepted)

### DNS (Domain Name System)

**Problem**: Hard to remember IP addresses
**Solution**: Domain names (like google.com)

**Analogy**: Phone book
- Name (Domain) → Number (IP Address)
- You call "Mom" → Phone dials actual number
- You type "google.com" → Connects to actual IP

**What DNS Does**:
- Maps domain names to IP addresses
- Directory/database service
- HTTP uses DNS to find IP address
- Then connects to server

#### DNS Structure

**Domain Name Parts**:
```
mail.google.com
└───┘ └────┘ └─┘
Subdomain | Top-Level
     Second-Level Domain (TLD)
```

**Subdomain**: mail (part of bigger domain)
**Second Level Domain**: google
**Top Level Domain**: .com, .org, .io, .edu

#### DNS Hierarchy

**1. Root DNS Servers**:
- First point of contact
- Top of hierarchy
- About 13 root server systems worldwide
- Check locations at: root-servers.org
- Operated by various organizations (Internet Systems Consortium, etc.)
- Have IPv4 and IPv6 addresses

**2. Top Level Domain (TLD) Servers**:
- .com, .org, .edu, .net, .io
- .uk, .us, .in (country-specific)
- Managed by **ICANN** (Internet Corporation for Assigned Names and Numbers)
- icann.org - registers all TLD domains
- Organization type specific:
  - .com: Commercial organizations
  - .edu: Educational institutions
  - .org: Non-profit organizations
- Now open to general use

**3. Second Level Domain**:
- google.com, amazon.com
- Your specific domain

#### DNS Resolution Process

**Steps When You Type google.com**:

```
1. Check Local Cache (your computer)
   - Visited before? IP stored locally
   - Fast, no need to search again

2. Local DNS Server (ISP)
   - Your ISP's DNS server
   - ISP knows ALL websites you visit (even incognito)
   - Can be required to report to police/government

3. Root Server
   - Ask: "Do you have .com?"
   - Root: "Ask TLD server"

4. TLD Server (.com server)
   - Has all .com domains
   - Returns: IP address of google.com

5. Connect to google.com Server
   - Using resolved IP address
```

**Command to Check DNS**:
```bash
dig google.com
# Shows DNS lookup information
# IP address in local cache
# DNS lookup utility
```

**Important Notes**:
- You cannot BUY domain names
- You can only RENT them
- Rent from: GoDaddy, Namecheap, etc.
- They pay ICANN, you pay them
- Some organizations have own TLDs:
  - Google has .google
  - Amazon owns .fire

### Client-Server Architecture

**Client**:
- Your device making requests
- Consumes resources
- Example: You accessing Google

**Server**:
- System hosting website/service
- Sends responses
- Characteristics:
  - Reliable IP address
  - High availability (always up)
  - High upload speed

**Data Centers**:
- Collection of many servers
- Large companies (Google, Facebook, Microsoft)
- Static IP addresses
- Very good internet connection
- Never goes down
- Cloud providers rent servers

**Why Many Servers**:
- One server can't handle millions of requests
- Load balancing across multiple servers
- Example: PS5 pre-order launch
  - Too many people
  - Servers crashed
- Example: Exam results
  - All students at once
  - Website crashes

**Testing with Ping**:
```bash
ping google.com

# Output shows:
- Packets sent/received (64 bytes each)
- Sequence numbers
- Time to Live (TTL): 60 (max hops)
- Round trip time (milliseconds)
- IP address of Google server
- 0 packet loss = all received
```

**TTL (Time to Live)**:
- Packet has hop limit
- Each router hop decreases TTL
- If reaches 0, packet dropped
- Prevents infinite loops

**Can We Reduce Ping Time?**:
- Not really
- Already at speed of light through cables
- Best possible time

### Peer-to-Peer (P2P) Architecture

**Characteristics**:
- No dedicated server
- All devices act as both client and server
- Decentralized network
- Devices connect directly

**Example**: College computers connected
```
Computer 1 ←→ Computer 2
    ↕           ↕
Computer 3 ←→ Computer 4
```

**Advantages**:
- Rapid scalability
- Decentralized
- No single point of failure

**Example**: BitTorrent
- File sharing
- Seeding concept
- Each peer shares parts of file

**Hybrid Models**: 
- Mix of P2P and centralized database
- Some are P2P, some centralized

### Application Layer Concepts

#### Programs, Processes, and Threads

**Program**:
- Your application
- Example: WhatsApp, Amazon, Gmail

**Process**:
- Running instance of program
- Feature of the program
- Example in WhatsApp:
  - Send message (process)
  - Record video (process)
  - One program, multiple processes

**Thread**:
- Lighter version of process
- Does one single job
- One process can have multiple threads
- Example sending message:
  - Set up page (thread)
  - Send data (thread)
- Multi-threading: Multiple threads working simultaneously

#### Sockets

**What Is a Socket**:
- Software process (not hardware)
- Interface between process and internet
- Gateway between application and network

**How Applications Communicate**:
- WhatsApp to WhatsApp
- Each has socket
- Socket has port number
- Applications communicate through sockets

---

## Layer 2: Transport Layer

### Overview

**Location**: On your devices (end systems)

**Main Purpose**: 
- **From network to application** (receiving)
- **From application to network** (sending)

**Key Concept**:
```
Network Layer: Computer A → Computer B (between devices)
Transport Layer: Within devices (network ↔ application)
```

### Understanding Transport Layer

**Analogy - Sending a Box**:
```
You → Courier Company (Your Country)
         ↓ (Transport Layer)
     Network Layer
         ↓
Courier Company (Friend's Country)
         ↓ (Transport Layer)
    Your Friend
```

**What It Does**:
1. Takes data from application
2. Gives to network layer (for sending)
3. Takes data from network layer  
4. Gives to correct application (for receiving)

**Example - Multiple Applications**:
```
Your Device:
- WhatsApp message
- Skype video call
- Gmail email

All go through network → Friend's device

Friend's Device:
- Which app gets message?
- Which app gets video?
- Which app gets email?

Transport Layer decides using PORTS!
```

### Multiplexing and Demultiplexing

**Multiplexing (Sender Side)**:
- Multiple applications → One network connection
- Like putting items in one box:
  - Cookies + PlayStation + Phone → One box

**Demultiplexing (Receiver Side)**:
- One network connection → Multiple applications
- Like opening box and distributing:
  - Box → Cookies, PlayStation, Phone to correct places

**How It Works**:
- Uses **port numbers**
- Each application has unique port
- Transport layer attaches port numbers
- Data travels as **segments**

### Key Functions

**1. Segmentation**:
- Divides data into small units (segments)
- Each segment has:
  - Source port number
  - Destination port number
  - Sequence number
  - Data

**2. Flow Control**:
- Controls amount of data transferred
- Example:
  - Server sending at 40 Mbps
  - Client receiving at 20 Mbps
  - Transport layer: "Slow down!"

**3. Error Control**:
- Checks if data packets lost
- Checks if data corrupted
- Uses checksums

**4. Congestion Control**:
- Network has lower bandwidth
- Sending packets too rapidly
- Some packets get lost
- Solution: Send at slower rate
- Uses congestion control algorithms

### Checksums

**Purpose**: Verify data not corrupted

**How It Works**:
```
Sender:
1. Have data
2. Calculate checksum using algorithm
3. Attach checksum to data
4. Send both

Receiver:
1. Receive data + checksum
2. Calculate checksum using same algorithm
3. Compare:
   - Same = data good ✓
   - Different = data corrupted ✗
```

### Timers (Retransmission Timer)

**Purpose**: Know if packets lost

**How It Works**:
```
Sender:
1. Send Packet 1
2. Start timer
3. Receiver sends: "Got it"
4. Stop timer ✓

If timer expires without "Got it":
- Packet probably lost
- Resend packet
```

**Problem - Duplicate Packets**:
```
1. Send Packet 2, start timer
2. Receiver gets it, sends "Got it"
3. But "Got it" message lost
4. Timer expires
5. Send Packet 2 again
6. Receiver now has 2 copies!
```

**Solution**: Sequence Numbers

### Sequence Numbers

**Purpose**: 
- Identify each packet uniquely
- Maintain order
- Detect duplicates

**Why Random**:
- Security
- Hard to guess
- Prevents hackers from hijacking connection

**How It Works**:
```
Packet 1: Sequence = 32
Packet 2: Sequence = 33
Packet 3: Sequence = 34

If receive: 32, 33, 33, 34
Know: 33 is duplicate!
```

### Transport Layer Protocols

#### UDP (User Datagram Protocol)

**Characteristics**:
- **Connectionless** (no connection established)
- **Stateless** (doesn't maintain state)
- Data may NOT be delivered
- Data may change on the way
- Data may NOT be in order

**When to Use**:
- Video conferencing (frames can drop)
- Gaming (some data loss okay)
- DNS (fast lookup needed)

**Advantages**:
- **Very fast** (no connection overhead)
- No acknowledgements
- Low latency

**UDP Packet Structure**:
```
Source Port (2 bytes)
Destination Port (2 bytes)
Length (2 bytes)
Checksum (2 bytes)
Data (up to 65,000 bytes)

Total Header: 8 bytes
Max Data: 2^16 - 8 = ~65,000 bytes
```

**Uses Checksum**: Yes (to detect corruption, but won't fix it)

**Command to See UDP Packets**:
```bash
tcpdump -c 5
# Shows 5 packets
# See UDP packets, sequence, length, IP addresses
```

#### TCP (Transmission Control Protocol)

**Characteristics**:
- **Connection-oriented** (establishes connection first)
- **Reliable** (all data delivered)
- **Ordered** (data arrives in sequence)
- **Error control**
- **Flow control**
- **Congestion control**

**When to Use**:
- HTTP/HTTPS (web browsing)
- Email (SMTP, POP3, IMAP)
- File transfers
- Any application needing 100% data delivery

**Features**:
1. **Connection-oriented**: Connection established before data transfer
2. **Full Duplex**: Both can send simultaneously
3. **Error Control**: Detects and fixes errors
4. **Congestion Control**: Manages network traffic
5. **Only 2 endpoints**: One TCP connection between two computers

**TCP Segment Structure** (More complex than UDP):
```
Source Port
Destination Port
Sequence Number
Acknowledgement Number
Flags (SYN, ACK, FIN, RST, etc.)
Window Size
Checksum
Urgent Pointer
Options
Data
```

### Three-Way Handshake

**Purpose**: Establish TCP connection

**Very Important**: Asked in interviews frequently!

**Steps**:
```
Client                          Server

1. SYN (Seq=32)        →
   "I want to connect"
   Sequence: Random (32)
   Start timer

2.                     ← SYN-ACK (Seq=56, Ack=33)
   "Sure, I accept"
   Sequence: Random (56)
   Ack: Client's Seq + 1

3. ACK (Seq=33, Ack=57) →
   "Acknowledged"
   Seq: Previous + 1
   Ack: Server's Seq + 1

✓ CONNECTION ESTABLISHED
```

**Why Random Sequence Numbers**:
- Security
- Hard to guess
- Prevents hackers from hijacking
- If predictable, easy to fake connection

**Flags Used**:
- **SYN** (Synchronize): New connection
- **ACK** (Acknowledge): Acknowledgement
- **FIN** (Finish): Close connection
- **RST** (Reset): Reset connection

**Math on Sequence**:
```
Client sends: Seq = 32
Server does math: 32 + calculation = 56
Server sends: Seq = 56, Ack = 33

Client next: Seq = 33 (32+1)
Ack = 57 (56+1)
```

---

## Layer 3: Network Layer

### Overview

**Main Purpose**: Transport data packets from one computer to another computer (possibly in different network)

**Location**: Routers work here

**Key Concept**:
```
Transport Layer: Within device (app ↔ network)
Network Layer: Between devices (computer ↔ computer)
```

**What It Does**:
1. Assigns IP addresses (sender and receiver)
2. Creates packets (from segments)
3. Routes packets through routers
4. Finds best path
5. Delivers to destination network

**Data Unit**: **Packets**

### Routing

**How It Works**:
```
Computer A                    Computer B
    |                             |
Router 1 - Router 2 - Router 3 - Router 4
    |         |         |         |
Router 5 - Router 6 - Router 7

Data hops from router to router until reaching destination
```

**Hop-by-Hop Forwarding**:
- Packet goes router to router
- Each router checks destination
- Forwards to next best router
- Continues until reaches destination

### Network Addresses

**IP Address Parts**:
```
192.168.2.30
└─────┘ └──┘
Network  Host
Address  Address
```

**Network Address**: Which network device is in
**Host Address**: Which device within that network

### Routing Tables and Forwarding Tables

**Routing Table**:
- Contains multiple paths to destination
- Stored in router
- Has all possible routes

**Forwarding Table**:
- Subset of routing table
- Only ONE path (fastest/best)
- Used for quick lookup
- Data structure in router

**How Router Works**:
```
1. Packet arrives at Router 1
2. Check destination IP
3. Look in forwarding table
4. "Destination 192.168.3.x is East"
5. Forward packet to Router 2 (East)
6. Router 2 repeats process
```

**Example**:
- Packet for 192.168.3.1
- Router checks: "Is this for me?" No
- Checks forwarding table: "Forward East"
- Sends to next router

### Routing Types

#### Static Routing
**Method**: Manually add routes
**Process**: Administrator adds each route by hand
**Problems**:
- Time-consuming
- Not adaptive
- If new router added, must update manually
- Hard to maintain

#### Dynamic Routing
**Method**: Routes adapt automatically
**Process**: Routers communicate, build routing tables
**Advantages**:
- Adapts to network changes
- Automatic updates
- Scalable

**Algorithms Used**:
- **Bellman-Ford** algorithm
- **Dijkstra's** shortest path algorithm
- **Graph theory** (routers = nodes, links = edges)

**Important**: Entire internet built on DSA (Data Structures & Algorithms)!

### Control Plane

**Purpose**: Builds routing tables

**What It Does**:
- Creates routing information
- Updates routing tables
- Manages network topology
- Routers = nodes in graph
- Links = edges in graph

### IP (Internet Protocol)

**What It Is**:
- Network layer protocol
- In TCP/IP model: The "IP" part
- Handles IP addressing
- Handles routing
- Manages packet delivery

### IP Addresses (IPv4)

**Format**: 4 numbers, each 0-255
```
192.168.1.1
```

**Binary Representation**:
```
192     .168     .1       .1
11000000.10101000.00000001.00000001
8 bits   8 bits   8 bits   8 bits = 32 bits
```

**What It Defines**:
- Server
- Client  
- Node
- Router

### Subnetting

**Problem**: Can't store every individual router's IP
**Solution**: Blocks of IP addresses

**How It Works**:
- ISPs get blocks of IPs
- Example: Airtel gets 192.168.x.x
- All devices in that network start with 192.168
- Router only needs to know network, not individual devices

**Subnet Example**:
```
192.168.1.0/24

192.168.1 = Network (first 24 bits)
.0 = Host (remaining 8 bits)

Can have: 192.168.1.0 to 192.168.1.255
Total: 256 addresses (2^8)
```

### IP Address Classes

**Class A**:
```
Range: 0.0.0.0 to 127.255.255.255
Network: First 8 bits
Host: Last 24 bits
```

**Class B**:
```
Range: 128.0.0.0 to 191.255.255.255
Network: First 16 bits
Host: Last 16 bits
```

**Class C**:
```
Range: 192.0.0.0 to 223.255.255.255
Network: First 24 bits
Host: Last 8 bits
```

**Class D**: 224.0.0.0 to 239.255.255.255
**Class E**: 240.0.0.0 to 255.255.255.255

### Subnet Masking

**Purpose**: Mask network portion, leave host portion

**Class C Example**:
```
Subnet Mask: 255.255.255.0
Meaning: First 3 numbers fixed (network)
         Last number variable (hosts)

Network: 255.255.255.x
Can use: .0 to .255 for hosts
```

**Variable Length Subnet**:
```
192.168.1.0/24
            ↑
First 24 bits = network
Remaining 8 bits = host
Total devices: 2^8 = 256

192.168.1.0/31
First 31 bits = network
Remaining 1 bit = host
Total devices: 2^1 = 2
```

### IP Address Allocation

**Initially**: First-come, first-served
- Large organizations got Class A
- MIT, Stanford, IBM got large blocks

**Currently**: Region-based
- **IETF** (Internet Engineering Task Force) assigns
- Based on regions, not classes
- Minimizes hops
- More efficient routing

### Reserved IP Addresses

**Loopback Addresses**:
```
127.0.0.0/8
Example: 127.0.0.1 (localhost)
```

**What It Does**:
- Computer communicates with itself
- Device acts as both client and server
- For testing purposes
- Always up if computer is running
- TCP/IP protocols can contact same processes

### IP Packets

**Header Size**: 20 bytes (minimum)

**Contains**:
- IP version (IPv4 or IPv6)
- Total length
- Identification number
- Flags
- Protocol (TCP, UDP, etc.)
- Source IP address
- Destination IP address
- **TTL (Time to Live)**
- Checksum
- Options
- Data

**TTL (Time to Live)**:
```
Initial: 64 (for example)
Each router hop: -1
If reaches 0: Packet dropped
```

**Why TTL**:
- Prevents infinite loops
- Packet doesn't roam forever
- If not delivered after X hops, drop it

**See TTL in Action**:
```bash
ping google.com

64 bytes from google.com: icmp_seq=0 ttl=60 time=10ms
# TTL=60 means can do 60 more hops
# Sequence number shown
# 0 packet loss = all received
```

### IPv6

**Size**: 128 bits (vs IPv4's 32 bits)
**Format**: 8 groups of hexadecimal
```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
or
abfe:f001:3210:9182:0001:0002:0003:0004
```

**Total Addresses**: 2^128 = 3.4 × 10^38

**Why Not Fully Adopted**:
- Not backward compatible with IPv4
- IPv4 devices can't access IPv6 servers
- ISPs need hardware upgrades
- Requires significant effort/cost

**Representation**:
```
Zeros can be compressed:
2001:0db8:0000:0000:0000:0000:0000:0001
Can write as:
2001:0db8::1

With prefix:
abfe:f001:3210:9182::/60
First 60 bits fixed, rest variable
```

### Middle Boxes

**What They Are**: Devices between computers and routers that interact with packets

#### Firewall

**Layers**: Network & Transport

**Types**:
1. **Stateless Firewall**:
   - Doesn't maintain state
   - Checks each packet independently
   
2. **Stateful Firewall**:
   - Maintains state/memory
   - Stores packet info in cache
   - More efficient
   - Remembers previous packets

**What It Does**:
- **Filter** packets (allow/block)
- **Modify** packets (change destination)
- Set rules on:
  - IP addresses (block specific IPs)
  - Port numbers (block specific ports)
  - Flags (block SYN to prevent new connections)
  - Protocols

**Location**:
- Between your network and internet
- On host machines
- In network infrastructure

#### NAT (Network Address Translation)

**Purpose**: Map one IP address space to another

**How It Works**:
```
Internal Network: 10.0.0.x (private IPs)
        ↓
      NAT
        ↓
External World: 150.150.0.1 (public IP)

All internal devices appear as 150.150.0.1 externally
```

**Why Use NAT**:
- Slow down consumption of IPv4 addresses
- Multiple devices share one public IP
- Security (hide internal IPs)

**What It Does**:
1. Modifies IP header
2. Changes source IP (internal → public)
3. Stores mapping in table
4. On return:
   - Checks table
   - Forwards to correct internal device

**Uses**:
- Routing
- Load balancing
- Enterprise networks

**Technical Details**:
- Replaces source address with public address
- Updates checksum
- Carried out via TCP or UDP
- Maintains translation table

---

## Layer 4: Data Link Layer

### Overview

**Main Purpose**: Send packets over physical link between directly connected devices

**Location**: Within LANs, between connected devices

**Key Concept**:
```
Network Layer: Computer A → Computer B (across networks)
Data Link Layer: Devices on same network (within LAN)
```

**Data Unit**: **Frames**

### What It Does

**Physical Addressing**:
- Network Layer uses IP (logical addressing)
- Data Link Layer uses MAC (physical addressing)
- Adds MAC addresses to create frames

**Frame Structure**:
```
MAC Source Address
MAC Destination Address  
IP Source Address (from packet)
IP Destination Address (from packet)
Data
Error Detection info
```

### DHCP (Dynamic Host Configuration Protocol)

**Purpose**: Automatically assign IP addresses

**Scenario**:
```
Router (DHCP Server)
    ├─ Device 1 (needs IP)
    ├─ Device 2 (needs IP)
    └─ Device 3 (needs IP)
```

**How It Works**:
```
1. New device connects to router
2. Device: "I need an IP address"
3. DHCP server has pool of IPs:
   192.168.1.2
   192.168.1.3
   192.168.1.4
   etc.
4. Server assigns one IP to device
5. Device now has IP address
```

**Where It Works**: Data Link Layer and Network Layer

### MAC Addresses

**What It Is**:
- **M**edia **A**ccess **C**ontrol address
- 12-digit alphanumeric string
- Example: `A1:B2:C3:D4:E5:F6`

**Important**:
- Not for the device itself
- For the network interface component
- Wi-Fi has different MAC than Bluetooth
- Each network interface has unique MAC

**Purpose**:
- Identify device on local network
- Physical addressing
- Used by all network technologies:
  - Ethernet
  - Wi-Fi
  - Bluetooth

### ARP (Address Resolution Protocol)

**Problem**: How to find MAC address from IP address?

**Scenario**:
```
Device 1 (192.168.1.2) wants to send to Device 4 (192.168.1.5)
But doesn't know Device 4's MAC address
```

**How It Works**:
```
1. Device 1 checks ARP cache
   "Do I have MAC for 192.168.1.5?" → No

2. Device 1 broadcasts ARP request to ALL devices:
   Frame contains:
   - Sender MAC address
   - Sender IP address
   - Target IP address: 192.168.1.5
   - Target MAC: (unknown)
   
3. ALL devices receive broadcast:
   - Device 2: "Not me, ignore"
   - Device 3: "Not me, ignore"
   - Device 4: "That's me!"
   
4. Device 4 responds:
   "My IP is 192.168.1.5, my MAC is XX:XX:XX:XX:XX:XX"
   
5. Device 1 updates ARP cache:
   192.168.1.5 → XX:XX:XX:XX:XX:XX
   
6. Future communication uses cached MAC
```

**ARP Cache**: Local storage of IP-to-MAC mappings

**Updates**: As new data received, cache updated

### Data Link Layer Functions

**1. Framing**:
- Converts packets to frames
- Adds MAC addresses

**2. Error Detection**:
- Checks for transmission errors
- Uses checksums

**3. Media Access Control**:
- Controls how data placed on media
- Controls how data received from media
- Techniques to get frame on/off physical medium

**4. Access to Upper Layers**:
- Allows OSI upper layers to access frames
- Passes frames up the stack

### How Router Uses MAC Addresses

**Process**:
```
1. Request from Device 1 goes to router
2. Request goes to internet
3. Response comes back to router
4. Router has private IP addresses mapped to MACs
5. Router forwards to correct device using MAC
```

**Why MAC Changes**:
- Public/private IP remains same
- MAC address changes as data travels
- Each physical hop uses different MAC addresses

**Command to Check MAC**:
```bash
ifconfig
# Shows network interfaces
# Each interface has MAC address
```

### Data Link Layer Devices

**Works At This Layer**:
- **Bridges**: Filter by MAC address
- **Switches**: Multi-port bridges, error checking

---

## Layer 5: Physical Layer

### Overview

**Main Purpose**: Convert data to electrical/light/radio signals

**What It Contains**:
- Hardware
- Physical medium (wires, cables)
- Mechanical components

**Data Unit**: **Bits** (0s and 1s)

### What It Does

**Transmission**:
1. Receives frames from Data Link Layer
2. Frames are in binary (0s and 1s)
3. Converts to appropriate signal:
   - **Electrical signals** (copper cables)
   - **Light signals** (optical fiber)
   - **Radio signals** (Wi-Fi, Bluetooth)

**Reception**:
1. Receives signals from physical medium
2. Converts signals to bits
3. Passes bits to Data Link Layer as frames
4. Frames move up the stack

**No Packets/Segments Here**: Only raw bits

### Physical Media

**Cables**:
- Ethernet cables
- Optical fiber cables
- Coaxial cables

**Wireless**:
- Radio waves (Wi-Fi)
- Microwaves (4G, 5G)
- Infrared

---

## TCP/IP Model

### Overview

**What It Is**:
- Internet Protocol Suite
- Developed by ARPA (remember ARPANET?)
- More practical than OSI model
- Actually used in real world

**Difference from OSI**:
- OSI: 7 layers (theoretical)
- TCP/IP: 5 layers (practical)

### The 5 Layers

```
5. Application Layer
   (Combines OSI's Application + Presentation + Session)

4. Transport Layer
   (Same as OSI)

3. Network Layer
   (Same as OSI)

2. Data Link Layer
   (Same as OSI)

1. Physical Layer
   (Same as OSI)
```

**Key Difference**:
- Top 3 OSI layers merged into 1 Application Layer
- More practical for implementation
- Used in actual internet

---

## Complete Data Flow Example

### Sending a WhatsApp Message

**Your Device**:
```
7. Application Layer (WhatsApp)
   - You type: "Hello Friend"
   ↓
6. Presentation Layer
   - Convert to binary
   - Encrypt message
   - Compress data
   ↓
5. Session Layer
   - Establish session with friend
   - Authentication
   ↓
4. Transport Layer
   - Divide into segments
   - Add source port: 5000
   - Add destination port: 5001
   - Add sequence numbers
   - Use TCP (need all data)
   ↓
3. Network Layer
   - Create packets
   - Add your IP: 192.168.1.2
   - Add friend's IP: 203.45.67.89
   - Check routing table
   ↓
2. Data Link Layer
   - Create frames
   - Add your MAC address
   - Add router's MAC address
   ↓
1. Physical Layer
   - Convert to electrical signals
   - Send through wire/Wi-Fi
```

**Through Internet**:
```
Physical Layer → Router 1
   ↓
Router checks Network Layer (IP address)
   ↓
Forwards to Router 2 (hop)
   ↓
Router 2 checks, forwards to Router 3 (hop)
   ↓
Continues hopping until reaches destination network
   ↓
Reaches friend's router
```

**Friend's Device**:
```
1. Physical Layer
   - Receives electrical signals
   - Converts to bits
   ↓
2. Data Link Layer
   - Reads frame
   - Checks MAC address
   - Passes to Network Layer
   ↓
3. Network Layer
   - Reads packet
   - Checks IP address
   - "This is for me!"
   - Passes to Transport Layer
   ↓
4. Transport Layer
   - Reads segment
   - Checks port: 5001 (WhatsApp)
   - Reassembles in order (sequence numbers)
   - Passes to Session Layer
   ↓
5. Session Layer
   - Manages session
   - Passes to Presentation Layer
   ↓
6. Presentation Layer
   - Decrypt message
   - Decompress
   - Convert from binary to text
   ↓
7. Application Layer (WhatsApp)
   - Display: "Hello Friend"
```

---

## Commands Reference

### Network Information
```bash
# Check your IP address
ifconfig
curl ifconfig.me

# Check MAC address
ifconfig

# Ping a server
ping google.com

# DNS lookup
dig google.com
nslookup -type=mx gmail.com

# See network packets (requires sudo)
tcpdump -c 5
```

### Browser DevTools
```
1. Open browser
2. Press F12 or Right-click → Inspect
3. Go to Network tab
4. Refresh page
5. See all requests/responses
6. Click any request to see:
   - Headers
   - Status codes
   - Data sent/received
```

---

## Important Interview Topics

### Must Know for Interviews

1. **OSI Model**:
   - All 7 layers
   - What each does
   - Examples for each

2. **TCP vs UDP**:
   - Differences
   - Use cases
   - Header structure

3. **Three-Way Handshake**:
   - Steps (SYN, SYN-ACK, ACK)
   - Why needed
   - Sequence numbers

4. **IP Addressing**:
   - IPv4 vs IPv6
   - Classes
   - Subnetting

5. **DNS Resolution**:
   - Complete process
   - Root, TLD, second-level domains

6. **HTTP**:
   - Methods (GET, POST, PUT, DELETE)
   - Status codes
   - Headers
   - Cookies

7. **Routing**:
   - Static vs Dynamic
   - Algorithms used
   - Routing tables

8. **NAT**:
   - What it does
   - Why needed
   - How it works

9. **MAC Addresses**:
   - What they are
   - ARP process

10. **Ports**:
    - Well-known ports
    - Categories
    - Why needed

---

## Key Takeaways

### Internet is NOT in the Cloud
- Physical cables under ocean
- Submarine cables connect continents
- Google owns many cables

### Layers Simplify Complexity
- Each layer has specific job
- Abstraction for developers
- Don't need to know everything to use internet

### Protocols are Rules
- Standardized communication
- Created by Internet Society
- Submitted via RFC process

### Data Transformation
```
Application: Message
Presentation: Binary, encrypted
Session: With session info
Transport: Segments (with ports)
Network: Packets (with IPs)
Data Link: Frames (with MACs)
Physical: Bits (electrical signals)
```

### Multiple Addresses
```
Device has:
- IP Address (logical, network layer)
- MAC Address (physical, data link layer)
- Port Numbers (application identifier)
```

### Everything is Built on DSA
- Routing uses Dijkstra's algorithm
- Graphs (routers = nodes, links = edges)
- Data structures everywhere

---

## Next Steps

### To Learn More
1. Research papers on protocols
2. Network certifications
3. Hands-on with Docker networking
4. Kubernetes networking
5. Cloud computing (AWS, GCP, Azure)

### Practice
1. Set up local server
2. Use tcpdump to see packets
3. Inspect browser network tab
4. Try creating own REST APIs
5. Experiment with different protocols

---

## Additional Resources

- **Internet Society**: internetsociety.org
- **ICANN**: icann.org
- **Submarine Cable Map**: submarinecablemap.com
- **Root Servers**: root-servers.org
- **RFC Documents**: ietf.org
- **First Website**: info.cern.ch/hypertext/www/project.html

---

*End of Computer Networking Course Notes*


