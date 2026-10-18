[← Back to all labs](../README.md)

# RIP Version 1 Dynamic Routing Across 4 LANs (Cisco Packet Tracer)

Four LANs, each behind its own router, are joined by three serial links in a chain. RIP version 1 is configured on every router so that all LANs learn each other's networks automatically, with no static routes.

## 📋 At a Glance

| | |
|---|---|
| **Lab number** | 5 |
| **Level** | Beginner, CCNA |
| **Tools** | Cisco Packet Tracer, Cisco IOS CLI |
| **Devices** | 4 routers (2811), 4 switches (2960), 8 PCs |
| **Protocols / features** | RIP version 1 (classful distance-vector routing), serial WAN links, IPv4 addressing |
| **Network(s)** | 192.168.10.0/24, 192.168.11.0/24, 192.168.12.0/24, 192.168.13.0/24 (LANs); 10.0.0.0/8, 11.0.0.0/8, 12.0.0.0/8 (serial links) |
| **CCNA domains** | IP Connectivity, Network Fundamentals |

## 📌 Project Objective

Configure RIPv1 on four routers (LAN-1 to LAN-4) so that every router learns the LAN and serial networks that are not directly connected to it. Success means each router's routing table holds `R` (RIP) entries for all remote networks, which lets a PC in one LAN reach a PC in any other LAN.

## 🌍 Real-World Scenario

Think of four small branch offices linked in a line by leased WAN circuits. Instead of an administrator typing a static route on every router each time a network is added, a dynamic routing protocol shares the routes between routers automatically. RIP is one of the oldest of these protocols. It is rarely chosen for new networks today (OSPF and EIGRP replaced it), but it is a common first step for learning how routers exchange routes, and the ideas it introduces (metric, administrative distance, update timers) carry over to every other protocol.

## 🖧 Topology

![Network topology showing four LANs connected in a chain by serial links](RIPv1.JPG)

Each LAN has one 2811 router, one 2960 switch and two PCs. The router's FastEthernet 0/0 connects to the switch's GigabitEthernet 0/1, and the two PCs connect to switch ports Fa0/1 and Fa0/2. The routers are connected in a chain with serial cables:

`LAN-1 ── LAN-2 ── LAN-3 ── LAN-4`

The topology shows the IP address of every router interface and every PC, plus the gateway written next to each PC.

### Cabling

| From | Port | To | Port | Cable type |
|---|---|---|---|---|
| LAN-1 router | Fa0/0 | LAN-1 switch | Gig0/1 | Copper straight-through |
| LAN-1 switch | Fa0/1 | LAN-1 PC (left) | Fa0 | Copper straight-through |
| LAN-1 switch | Fa0/2 | LAN-1 PC (bottom) | Fa0 | Copper straight-through |
| LAN-2 router | Fa0/0 | LAN-2 switch | Gig0/1 | Copper straight-through |
| LAN-2 switch | Fa0/1 | LAN-2 PC (left) | Fa0 | Copper straight-through |
| LAN-2 switch | Fa0/2 | LAN-2 PC (bottom) | Fa0 | Copper straight-through |
| LAN-3 router | Fa0/0 | LAN-3 switch | Gig0/1 | Copper straight-through |
| LAN-3 switch | Fa0/1 | LAN-3 PC (left) | Fa0 | Copper straight-through |
| LAN-3 switch | Fa0/2 | LAN-3 PC (bottom) | Fa0 | Copper straight-through |
| LAN-4 router | Fa0/0 | LAN-4 switch | Gig0/1 | Copper straight-through |
| LAN-4 switch | Fa0/1 | LAN-4 PC (right) | Fa0 | Copper straight-through |
| LAN-4 switch | Fa0/2 | LAN-4 PC (bottom) | Fa0 | Copper straight-through |
| LAN-1 router | Se0/0/0 | LAN-2 router | Se0/0/1 | Serial (DCE end on LAN-1) |
| LAN-2 router | Se0/0/0 | LAN-3 router | Se0/0/1 | Serial (DCE end on LAN-2) |
| LAN-3 router | Se0/0/0 | LAN-4 router | Se0/0/1 | Serial (DCE end on LAN-3) |

Cable types follow the standard rule: unlike devices (router to switch, switch to PC) use straight-through copper, and serial links use serial cables. In the topology, the clock icon sits on each `Se0/0/0` end, which marks the DCE side that supplies the clock. The port label for the LAN-3 left PC is partly hidden in the image; Fa0/1 is assumed from the pattern in the other LANs.

### Addressing Table

| Device | Interface | IP Address | Subnet Mask | Prefix | Default Gateway (as labelled in topology) |
|---|---|---|---|---|---|
| LAN-1 | Fa0/0 | 192.168.10.1 | 255.255.255.0 | /24 | N/A |
| LAN-1 | Se0/0/0 | 10.0.0.1 | 255.0.0.0 | /8 | N/A |
| LAN-2 | Fa0/0 | 192.168.11.1 | 255.255.255.0 | /24 | N/A |
| LAN-2 | Se0/0/1 | 10.0.0.2 | 255.0.0.0 | /8 | N/A |
| LAN-2 | Se0/0/0 | 11.0.0.1 | 255.0.0.0 | /8 | N/A |
| LAN-3 | Fa0/0 | 192.168.12.1 | 255.255.255.0 | /24 | N/A |
| LAN-3 | Se0/0/1 | 11.0.0.2 | 255.0.0.0 | /8 | N/A |
| LAN-3 | Se0/0/0 | 12.0.0.1 | 255.0.0.0 | /8 | N/A |
| LAN-4 | Fa0/0 | 192.168.13.1 | 255.255.255.0 | /24 | N/A |
| LAN-4 | Se0/0/1 | 12.0.0.2 | 255.0.0.0 | /8 | N/A |
| PC, LAN-1 (Fa0/1) | Fa0 | 192.168.10.2 | 255.255.255.0 | /24 | 10.0.0.1 |
| PC, LAN-1 (Fa0/2) | Fa0 | 192.168.10.3 | 255.255.255.0 | /24 | 10.0.0.1 |
| PC, LAN-2 (Fa0/1) | Fa0 | 192.168.11.2 | 255.255.255.0 | /24 | 10.0.0.2 |
| PC, LAN-2 (Fa0/2) | Fa0 | 192.168.11.3 | 255.255.255.0 | /24 | 11.0.0.1 |
| PC, LAN-3 (Fa0/1) | Fa0 | 192.168.12.2 | 255.255.255.0 | /24 | 11.0.0.2 |
| PC, LAN-3 (Fa0/2) | Fa0 | 192.168.12.3 | 255.255.255.0 | /24 | 12.0.0.1 |
| PC, LAN-4 (Fa0/1) | Fa0 | 192.168.13.3 | 255.255.255.0 | /24 | 12.0.0.2 |
| PC, LAN-4 (Fa0/2) | Fa0 | 192.168.13.2 | 255.255.255.0 | /24 | 12.0.0.2 |

The `/8` mask on the serial interfaces is shown in the LAN-2, LAN-3 and LAN-4 routing-table screenshots (`10.0.0.0/8`, `11.0.0.0/8`, `12.0.0.0/8` directly connected). The LAN-1 serial mask is taken from the other end of the same link.

**Note on the PC gateways:** the topology labels each PC's gateway with a router *serial* address (for example 10.0.0.1), not with the router's LAN address (for example 192.168.10.1). A host's default gateway is normally an address inside the host's own subnet. The table above records the gateways exactly as labelled in the topology image.

### Subnet Analysis

| Network | Mask | Network Address | First Usable | Last Usable | Broadcast | Usable Hosts | Private (RFC 1918)? |
|---|---|---|---|---|---|---|---|
| LAN-1 | /24 | 192.168.10.0 | 192.168.10.1 | 192.168.10.254 | 192.168.10.255 | 254 | Yes |
| LAN-2 | /24 | 192.168.11.0 | 192.168.11.1 | 192.168.11.254 | 192.168.11.255 | 254 | Yes |
| LAN-3 | /24 | 192.168.12.0 | 192.168.12.1 | 192.168.12.254 | 192.168.12.255 | 254 | Yes |
| LAN-4 | /24 | 192.168.13.0 | 192.168.13.1 | 192.168.13.254 | 192.168.13.255 | 254 | Yes |
| Link LAN-1 to LAN-2 | /8 | 10.0.0.0 | 10.0.0.1 | 10.255.255.254 | 10.255.255.255 | 16,777,214 | Yes (10.0.0.0/8) |
| Link LAN-2 to LAN-3 | /8 | 11.0.0.0 | 11.0.0.1 | 11.255.255.254 | 11.255.255.255 | 16,777,214 | No |
| Link LAN-3 to LAN-4 | /8 | 12.0.0.0 | 12.0.0.1 | 12.255.255.254 | 12.255.255.255 | 16,777,214 | No |

- Every LAN router interface uses the `.1` address, the first usable address in its subnet.
- Each serial link needs only two addresses, yet a /8 mask reserves over 16 million. A /30 is the usual choice for point-to-point links. Here the /8 is the natural mask of these Class A networks, which suits RIPv1 because it is classful.
- 11.0.0.0/8 and 12.0.0.0/8 are outside the RFC 1918 private ranges. That is fine inside a Packet Tracer simulation, but those addresses should not be used on a real network.

## 📚 Concepts Behind This Lab

**Dynamic routing with RIP.** RIP is a distance-vector protocol. Each router sends its whole routing table to its neighbours every 30 seconds, and a router learns routes only from what its neighbours tell it. RIP's metric is hop count: each router crossed adds 1, and the maximum is 15. A route with a metric of 16 is treated as unreachable. RIP has an administrative distance of 120, so a static route (AD 1) or an EIGRP/OSPF route would be preferred over it.

**What "version 1" means.** RIPv1 is classful. Its updates do not carry subnet masks, so a router assumes the standard class mask for a network it learns. It sends updates as broadcasts (255.255.255.255) and has no authentication. In this lab that is fine, because every network is used with its natural classful mask and no network is split up.

| | RIPv1 | RIPv2 |
|---|---|---|
| Classful / classless | Classful (no mask in updates) | Classless (mask in updates, supports VLSM and CIDR) |
| Update address | Broadcast 255.255.255.255 | Multicast 224.0.0.9 |
| Authentication | No | Yes |
| Auto-summary | Always on | On by default, can be turned off with `no auto-summary` |

**The `network` command.** In RIP, `network` takes a classful network number (10.0.0.0, 11.0.0.0, 192.168.10.0). It does two things at once: it enables RIP on every interface that falls inside that network, and it tells RIP to advertise that network. A router only needs `network` statements for its own directly connected networks, not for remote ones.

**`version 1`.** Without this command, a Cisco router running RIP sends version 1 updates but accepts both version 1 and version 2. Typing `version 1` makes the router send and accept only version 1.

**Serial links and clocking.** On a serial link one end is DCE and supplies the clock (`clock rate`), and the other end is DTE. If the DCE end has no clock rate, the link stays down. In Packet Tracer the DCE end is shown by a small clock icon.

**Reading the routing table.** `C` is a directly connected network, `L` is the router's own address (a /32), and `R` is a route learned through RIP, shown as `[120/hops]`.

## 🧭 Lab Steps

1. Build the topology: four 2811 routers, four 2960 switches and eight PCs, with the cabling shown above.
2. Assign IP addresses to the router LAN interfaces, the serial interfaces and the PCs, as in the addressing table.
3. Check the baseline routing table on a router (LAN-2, LAN-3 and LAN-4 screenshots): only connected (`C`) and local (`L`) routes appear.
4. On each router, enter RIP configuration mode, set version 1, and add a `network` statement for each directly connected network.
5. Verify that each router learns the remote networks and that PCs in different LANs can reach each other (see Verification).

## ⚙️ Configuration

### Step 1: Interface addressing and baseline routing tables

The interface IP addresses and PC settings are shown in the topology image above. The commands used to set them were not part of the screenshots, so they are not reproduced as performed work. The LAN-2, LAN-3 and LAN-4 screenshots each begin with a `show ip route` taken before RIP was configured. The interfaces carry the addresses from the table (`C` and `L` entries), and no `R` routes exist yet.

Baseline from LAN-2 (before RIP):

```
LAN-2# show ip route
...
C    10.0.0.0/8 is directly connected, Serial0/0/1
L    10.0.0.2/32 is directly connected, Serial0/0/1
C    11.0.0.0/8 is directly connected, Serial0/0/0
L    11.0.0.1/32 is directly connected, Serial0/0/0
C    192.168.11.0/24 is directly connected, FastEthernet0/0
L    192.168.11.1/32 is directly connected, FastEthernet0/0
```

**Notes:** `L` entries are the router's own interface addresses. Because they are /32 host routes, IOS prints each parent network as "variably subnetted, 2 subnets, 2 masks" even though only one real mask is in use.

### Step 2: Configure RIPv1 on LAN-1

![LAN-1 CLI showing RIP version 1 configured with networks 10.0.0.0 and 192.168.10.0](LAN-1_RIPv1_Config.JPG)

```
LAN-1> enable
LAN-1# configure terminal
LAN-1(config)# router rip
LAN-1(config-router)# version 1
LAN-1(config-router)# network 10.0.0.0
LAN-1(config-router)# network 192.168.10.0
LAN-1(config-router)# exit
```

| Command | Mode | What it does |
|---|---|---|
| `enable` | User EXEC | Enters privileged EXEC mode (prompt changes from `>` to `#`) |
| `configure terminal` | Privileged EXEC | Enters global configuration mode |
| `router rip` | Global config | Starts the RIP process and enters router configuration mode (`(config-router)#`) |
| `version 1` | Router config | Sends and accepts RIP version 1 only |
| `network 10.0.0.0` | Router config | Enables RIP on the serial link to LAN-2 and advertises 10.0.0.0 |
| `network 192.168.10.0` | Router config | Enables RIP on Fa0/0 and advertises the LAN-1 network |
| `exit` | Router config | Returns to global configuration mode |

**Notes:** the console banner at the top of this screenshot ("con0 is now available") is the normal message when a device CLI opens. LAN-1 is the end of the chain, so it has only two directly connected networks.

### Step 3: Configure RIPv1 on LAN-2

![LAN-2 CLI showing show ip route output and RIP version 1 configuration](LAN-2_RIPv1_Config.JPG)

```
LAN-2# show ip route
LAN-2# configure terminal
LAN-2(config)# router rip
LAN-2(config-router)# version 1
LAN-2(config-router)# network 10.0.0.0
LAN-2(config-router)# network 11.0.0.0
LAN-2(config-router)# network 192.168.11.0
LAN-2(config-router)# exit
```

| Command | Mode | What it does |
|---|---|---|
| `show ip route` | Privileged EXEC | Displays the routing table (baseline before RIP) |
| `configure terminal` | Privileged EXEC | Enters global configuration mode |
| `router rip` | Global config | Starts the RIP process |
| `version 1` | Router config | Restricts RIP to version 1 |
| `network 10.0.0.0` | Router config | Enables RIP on Se0/0/1 (link to LAN-1) |
| `network 11.0.0.0` | Router config | Enables RIP on Se0/0/0 (link to LAN-3) |
| `network 192.168.11.0` | Router config | Enables RIP on Fa0/0 (LAN-2 network) |
| `exit` | Router config | Returns to global configuration mode |

**Notes:** LAN-2 sits in the middle of the chain with three connected networks. It passes routes in both directions, so LAN-1 learns about LAN-3 and LAN-4 through it.

### Step 4: Configure RIPv1 on LAN-3

![LAN-3 CLI showing show ip route output and RIP version 1 configuration](LAN-3_RIPv1_Config.JPG)

```
LAN-3# show ip route
LAN-3# configure terminal
LAN-3(config)# router rip
LAN-3(config-router)# version 1
LAN-3(config-router)# network 11.0.0.0
LAN-3(config-router)# network 12.0.0.0
LAN-3(config-router)# network 192.168.12.0
LAN-3(config-router)# exit
```

| Command | Mode | What it does |
|---|---|---|
| `show ip route` | Privileged EXEC | Displays the routing table (baseline before RIP) |
| `router rip` | Global config | Starts the RIP process |
| `version 1` | Router config | Restricts RIP to version 1 |
| `network 11.0.0.0` | Router config | Enables RIP on Se0/0/1 (link to LAN-2) |
| `network 12.0.0.0` | Router config | Enables RIP on Se0/0/0 (link to LAN-4) |
| `network 192.168.12.0` | Router config | Enables RIP on Fa0/0 (LAN-3 network) |

### Step 5: Configure RIPv1 on LAN-4

![LAN-4 CLI showing show ip route output and RIP version 1 configuration](LAN-4_RIPv1_Config.JPG)

```
LAN-4> enable
LAN-4# show ip route
LAN-4# configure terminal
LAN-4(config)# router rip
LAN-4(config-router)# version 1
LAN-4(config-router)# network 12.0.0.0
LAN-4(config-router)# network 192.168.13.0
LAN-4(config-router)# exit
```

| Command | Mode | What it does |
|---|---|---|
| `show ip route` | Privileged EXEC | Displays the routing table (baseline before RIP) |
| `router rip` | Global config | Starts the RIP process |
| `version 1` | Router config | Restricts RIP to version 1 |
| `network 12.0.0.0` | Router config | Enables RIP on Se0/0/1 (link to LAN-3) |
| `network 192.168.13.0` | Router config | Enables RIP on Fa0/0 (LAN-4 network) |

**Notes:** the commands above were typed in abbreviated form in the screenshots (`conf t`, `ver 1`, `net`, `sh ip route`). They are written out in full here for readability. The configuration was not saved to NVRAM in any screenshot; `copy running-config startup-config` would keep it after a reload.

## ✅ Verification & Results

| Check | Command / Action | What success looks like | Result |
|---|---|---|---|
| Interfaces carry the planned addresses | `show ip route` (before RIP) | `C` and `L` entries for each interface network | Shown in screenshot (LAN-2, LAN-3, LAN-4) |
| RIP is configured on all routers | `router rip`, `version 1`, `network ...` | Commands accepted without errors | Shown in screenshot (LAN-1 to LAN-4) |
| Remote networks learned | `show ip route rip` | `R` entries with `[120/hops]` for every remote network | Expected (not captured) |
| RIP settings and neighbours | `show ip protocols` | Version 1, `network` list, routing information sources | Expected (not captured) |
| End-to-end connectivity | `ping` from a PC in one LAN to a PC in another | Replies from the far PC | Expected (not captured) |
| Path across the routers | `tracert` from a PC | One line per router hop | Expected (not captured) |

No screenshot in this lab shows the routing table after RIP was enabled, or any ping result. The results marked "Expected" are what a working configuration should produce.

**Expected output (not captured):** `show ip route rip` on LAN-1, with the timers shown as `xx`:

```
R    11.0.0.0/8 [120/1] via 10.0.0.2, 00:00:xx, Serial0/0/0
R    12.0.0.0/8 [120/2] via 10.0.0.2, 00:00:xx, Serial0/0/0
R    192.168.11.0/24 [120/1] via 10.0.0.2, 00:00:xx, Serial0/0/0
R    192.168.12.0/24 [120/2] via 10.0.0.2, 00:00:xx, Serial0/0/0
R    192.168.13.0/24 [120/3] via 10.0.0.2, 00:00:xx, Serial0/0/0
```

**Expected RIP routes on every router (hop counts worked out from the topology):**

| Router | Learned via RIP (hops) |
|---|---|
| LAN-1 | 11.0.0.0/8 (1), 192.168.11.0/24 (1), 12.0.0.0/8 (2), 192.168.12.0/24 (2), 192.168.13.0/24 (3) |
| LAN-2 | 192.168.10.0/24 (1), 12.0.0.0/8 (1), 192.168.12.0/24 (1), 192.168.13.0/24 (2) |
| LAN-3 | 10.0.0.0/8 (1), 192.168.11.0/24 (1), 192.168.13.0/24 (1), 192.168.10.0/24 (2) |
| LAN-4 | 11.0.0.0/8 (1), 192.168.12.0/24 (1), 10.0.0.0/8 (2), 192.168.11.0/24 (2), 192.168.10.0/24 (3) |

The farthest network from any router is 3 hops away, well inside RIP's 15-hop limit.

## 🛠️ Troubleshooting Notes

No problems were reported for this lab. The table below is general guidance for RIPv1 labs, not a record of issues hit.

| Symptom | Likely cause | How to check | Fix |
|---|---|---|---|
| No `R` routes at all | RIP not enabled on the link, or interface down | `show ip interface brief`, `show ip protocols` | `no shutdown` on the interface, add the missing `network` statement |
| Serial interface up/down | Missing clock rate on the DCE end, or encapsulation mismatch | `show controllers serial 0/0/0`, `show interfaces serial 0/0/0` | Set `clock rate` on the DCE end |
| A neighbour's networks are missing | Typo in a `network` statement, or the wrong network entered | `show running-config \| section router rip` | Remove the wrong line with `no network ...` and add the correct one |
| Routes learned but pings fail | Wrong PC IP, mask or default gateway | `ipconfig /all` on the PC, then ping the gateway first | Correct the PC settings |
| Ping works inside a LAN but not across LANs | Gateway not set correctly, or return route missing | `ping` the gateway, then `tracert` to see where it stops | Fix the gateway, check RIP on the router that stops replying |
| Route disappears after a few minutes | A neighbour stopped sending updates (invalid timer 180 s, flush 240 s) | `show ip route`, `debug ip rip` (stop with `undebug all`) | Restore the link or the neighbour's RIP configuration |

Work from the bottom up: cables and interfaces first, then IP settings, then routing.

## 💼 Skills Demonstrated

- Configured RIP version 1 on four Cisco routers using `router rip`, `version 1` and classful `network` statements.
- Planned and documented an IPv4 scheme with four /24 LANs and three serial point-to-point networks.
- Built a multi-router topology with serial WAN links, including DCE clocking.
- Read a Cisco routing table and told connected (`C`), local (`L`) and RIP (`R`) routes apart.
- Applied RIP concepts: hop-count metric, 15-hop limit, administrative distance 120, 30-second updates.
- Compared RIPv1 and RIPv2 and explained the limits of classful routing.
- Used a layered troubleshooting approach for routing problems.

**CCNA 200-301 mapping:** Domain 3.0 IP Connectivity (interpreting the routing table, how a router makes a forwarding decision, administrative distance and metric); Domain 1.0 Network Fundamentals (IPv4 addressing, subnetting, cabling and interface verification). RIP itself is not a hands-on exam topic in 200-301, but this lab builds the dynamic-routing foundation that OSPF builds on.

## 📂 Files in This Lab

```
5_RIP LAB/
├── README.md
├── RIPv1.JPG                  # Topology with IP addressing
├── LAN-1_RIPv1_Config.JPG     # LAN-1 router CLI, RIPv1 configuration
├── LAN-2_RIPv1_Config.JPG     # LAN-2 router CLI, baseline route table and RIPv1 configuration
├── LAN-3_RIPv1_Config.JPG     # LAN-3 router CLI, baseline route table and RIPv1 configuration
├── LAN-4_RIPv1_Config.JPG     # LAN-4 router CLI, baseline route table and RIPv1 configuration
└── PKT File/
    └── RIP_V1.pkt             # Packet Tracer lab file
```

## 🚀 How to Open This Lab

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco NetAcad account).
2. Open `PKT File/RIP_V1.pkt`.
3. Click a router (LAN-1 to LAN-4), open the **CLI** tab, press Enter, then run `enable` and `show ip route` to see the routing table. Run `show ip protocols` to see the RIP settings.
4. To test connectivity, click a PC, open the **Desktop** tab, then **Command Prompt**, and run `ping` to a PC in another LAN.

## 🔭 Possible Improvements

- Move to RIPv2 with `no auto-summary` so the network supports VLSM and carries masks in updates.
- Use `passive-interface fastEthernet 0/0` on each router so RIP updates are not sent into the LANs.
- Re-address the serial links as /30 networks to save addresses.
- Set each PC's gateway to its own router's LAN address and capture ping and `tracert` screenshots as evidence.
- Add a default route and `default-information originate` to simulate an internet connection.

## 👤 Author

[Your Name] — Aspiring Network Engineer, building CCNA / networking fundamentals skills.

---
*This lab is part of my ongoing CCNA / Network Administrator study roadmap.*
