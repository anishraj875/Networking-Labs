[← Back to all labs](../README.md)

# Static Routing Across a Multi-Router WAN, Six LANs in Two Buildings (Cisco Packet Tracer)

This lab connects six LANs across two buildings through four routers and four serial WAN links. Every router uses manually configured static routes so that every LAN can reach every other LAN. It demonstrates IPv4 addressing, subnetting, and static route design.

## 📋 At a Glance

| | |
|---|---|
| **Lab number** | [N] |
| **Level** | Beginner / Intermediate, CCNA |
| **Tools** | Cisco Packet Tracer, Cisco IOS CLI |
| **Devices** | 4 routers (R1 to R4), 6 switches (SW1 to SW6), 35 PCs |
| **Protocols / features** | Static IPv4 routing, Ethernet, serial WAN links, ICMP (ping) |
| **Network(s)** | LAN: 192.168.10.0/24, 192.168.20.0/24, 192.168.30.0/24, 192.168.40.0/24, 192.168.50.0/24, 192.168.60.0/24. WAN: 172.16.0.0/30, 172.16.0.4/30, 172.16.0.8/30 |
| **CCNA domains** | 3.0 IP Connectivity, 1.0 Network Fundamentals |

## 📌 Project Objective

Configure the interfaces on all four routers and add static routes so that a PC in any LAN can reach a PC in any other LAN. Success means that every router has a route to every remote network, and that pings between buildings succeed in both directions.

## 🌍 Real-World Scenario

Organisations with several buildings often connect them with leased lines or point-to-point links. When the topology is small and changes rarely, network engineers prefer static routes because they add no routing protocol traffic, are simple to predict, and are easy to audit. The same design is common on small branch and campus networks. The main drawback is that every route must be added by hand, and a failed link is not rerouted automatically. Dynamic protocols such as OSPF or EIGRP handle that job on larger networks.

## 🖧 Topology

![Network topology](Capture.JPG)

The topology has two buildings. **Building A** holds three LANs (A1, A2 and A3) attached to SW1, SW2 and SW3. **Building B** holds three LANs (B1, B2 and B3) attached to SW4, SW5 and SW6. Each switch connects to a router through its Gig0/1 port, and the routers are joined by serial links in a ring: R1 to R2, R2 to R4, and R4 to R3. R1 sits at the top of the shared area, R2 at the left, R4 at the right and R3 at the top right.

### Cabling

| From | Port | To | Port | Cable type |
|---|---|---|---|---|
| R1 | Gig0/0 | SW1 | Gig0/1 | Straight-through (copper) |
| R2 | Gig0/0 | SW2 | Gig0/1 | Straight-through (copper) |
| R2 | Gig0/1 | SW3 | Gig0/1 | Straight-through (copper) |
| R3 | Gig0/0 | SW4 | Gig0/1 | Straight-through (copper) |
| R4 | Gig0/0 | SW5 | Gig0/1 | Straight-through (copper) |
| R4 | Gig0/1 | SW6 | Gig0/1 | Straight-through (copper) |
| R1 | Se0/0/0 | R2 | Se0/0/0 | Serial (DCE/DTE) |
| R2 | Se0/0/1 | R4 | Se0/0/1 | Serial (DCE/DTE) |
| R4 | Se0/0/0 | R3 | Se0/0/1 | Serial (DCE/DTE) |
| PCs in each LAN | Fa0 | Fa0/3, Fa0/6 and other access ports on their switch | | Straight-through (copper) |

Routers and switches are different kinds of device, so the Ethernet links use straight-through cables. Serial links are dedicated WAN cables. One end of each serial cable is DCE and needs a `clock rate` command. See the Configuration section for details.

### Addressing Table

| Device | Interface | IP Address | Subnet Mask | Prefix | Default Gateway |
|---|---|---|---|---|---|
| R1 | Gig0/0 | 192.168.10.1 | 255.255.255.0 | /24 | n/a (router) |
| R1 | Se0/0/0 | 172.16.0.1 | 255.255.255.252 | /30 | n/a (router) |
| R2 | Gig0/0 | 192.168.20.1 | 255.255.255.0 | /24 | n/a (router) |
| R2 | Gig0/1 | 192.168.30.1 | 255.255.255.0 | /24 | n/a (router) |
| R2 | Se0/0/0 | 172.16.0.2 | 255.255.255.252 | /30 | n/a (router) |
| R2 | Se0/0/1 | 172.16.0.5 | 255.255.255.252 | /30 | n/a (router) |
| R3 | Gig0/0 | 192.168.40.1 | 255.255.255.0 | /24 | n/a (router) |
| R3 | Se0/0/1 | 172.16.0.10 | 255.255.255.252 | /30 | n/a (router) |
| R4 | Gig0/0 | 192.168.50.1 | 255.255.255.0 | /24 | n/a (router) |
| R4 | Gig0/1 | 192.168.60.1 | 255.255.255.0 | /24 | n/a (router) |
| R4 | Se0/0/0 | 172.16.0.9 | 255.255.255.252 | /30 | n/a (router) |
| R4 | Se0/0/1 | 172.16.0.6 | 255.255.255.252 | /30 | n/a (router) |
| PCs, LAN A1 | Fa0 | 192.168.10.2 to 192.168.10.7 | 255.255.255.0 | /24 | 192.168.10.1 |
| PCs, LAN A2 | Fa0 | 192.168.20.2 to 192.168.20.7 | 255.255.255.0 | /24 | 192.168.20.1 |
| PCs, LAN A3 | Fa0 | 192.168.30.2 to 192.168.30.6 | 255.255.255.0 | /24 | 192.168.30.1 |
| PCs, LAN B1 | Fa0 | 192.168.40.2 to 192.168.40.7 | 255.255.255.0 | /24 | 192.168.40.1 |
| PCs, LAN B2 | Fa0 | 192.168.50.2 to 192.168.50.7 | 255.255.255.0 | /24 | 192.168.50.1 |
| PCs, LAN B3 | Fa0 | 192.168.60.2 to 192.168.60.7 | 255.255.255.0 | /24 | 192.168.60.1 |

The PC addresses are the ones labelled on the topology image. The LAN A3 label shows hosts .2 to .6 only.

### Subnet Analysis

| Network | Mask | Network Address | First Usable | Last Usable | Broadcast | Usable Hosts | Private (RFC 1918)? |
|---|---|---|---|---|---|---|---|
| 192.168.10.0/24 | 255.255.255.0 | 192.168.10.0 | 192.168.10.1 | 192.168.10.254 | 192.168.10.255 | 254 | Yes |
| 192.168.20.0/24 | 255.255.255.0 | 192.168.20.0 | 192.168.20.1 | 192.168.20.254 | 192.168.20.255 | 254 | Yes |
| 192.168.30.0/24 | 255.255.255.0 | 192.168.30.0 | 192.168.30.1 | 192.168.30.254 | 192.168.30.255 | 254 | Yes |
| 192.168.40.0/24 | 255.255.255.0 | 192.168.40.0 | 192.168.40.1 | 192.168.40.254 | 192.168.40.255 | 254 | Yes |
| 192.168.50.0/24 | 255.255.255.0 | 192.168.50.0 | 192.168.50.1 | 192.168.50.254 | 192.168.50.255 | 254 | Yes |
| 192.168.60.0/24 | 255.255.255.0 | 192.168.60.0 | 192.168.60.1 | 192.168.60.254 | 192.168.60.255 | 254 | Yes |
| 172.16.0.0/30 | 255.255.255.252 | 172.16.0.0 | 172.16.0.1 | 172.16.0.2 | 172.16.0.3 | 2 | Yes (172.16.0.0/12) |
| 172.16.0.4/30 | 255.255.255.252 | 172.16.0.4 | 172.16.0.5 | 172.16.0.6 | 172.16.0.7 | 2 | Yes (172.16.0.0/12) |
| 172.16.0.8/30 | 255.255.255.252 | 172.16.0.8 | 172.16.0.9 | 172.16.0.10 | 172.16.0.11 | 2 | Yes (172.16.0.0/12) |

Each LAN uses the first usable address (.1) as its default gateway. Each WAN link uses a /30, which has exactly two usable addresses, one for each router on the link. The LAN addresses are not summarised, so each remote LAN needs its own static route.

## 📚 Concepts Behind This Lab

**Static routes.** A static route is a manual entry in the routing table, with the syntax `ip route <network> <mask> <next-hop>`. The router uses it to forward packets for that network to the next-hop router. This lab uses next-hop addresses rather than exit interfaces. A next-hop route is the better choice on a point-to-point serial link, because the router knows exactly which neighbour receives the packet.

**Route selection.** When a router has several routes that match a destination, it uses the longest prefix match first. A static route has an administrative distance (AD) of 1, while a directly connected network has an AD of 0. Connected routes are therefore preferred over static routes for the same prefix, and static routes are preferred over RIP (AD 120) and OSPF (AD 110).

| Route source | Administrative distance |
|---|---|
| Directly connected | 0 |
| Static | 1 |
| EIGRP internal | 90 |
| OSPF | 110 |
| RIP | 120 |

**Routing is two-way.** A ping from a PC in LAN A1 to a PC in LAN B3 needs two things: a route on every router in the path that leads to 192.168.60.0/24, and a route back to 192.168.10.0/24 on every router in the reply path. A static route in only one direction causes ping failures that look like a problem with the destination. Each router must have an entry for every LAN that is not directly connected to it.

**WAN links and clock rate.** Serial links connect routers through a Data Communications Equipment (DCE) end and a Data Terminal Equipment (DTE) end. The DCE side provides the clock signal and must set `clock rate` on the interface. Without it, the link may show up/down. Both ends of a link use addresses from the same /30, which gives two usable hosts: one for each router.

## 🧭 Lab Steps

1. Build the topology in Packet Tracer with four routers, six switches and 35 PCs, and connect the cables as shown in the Cabling table.
2. Set the hostname on each router and turn off DNS lookup.
3. Configure the IP address on each router interface, then enable it with `no shutdown`.
4. Set the IP address and default gateway on each PC.
5. Configure static routes on each router so that every remote network is reachable.
6. Save each router's configuration with `write memory`.
7. Verify the routing tables, interface status, and end-to-end pings.

## ⚙️ Configuration

### Step 1: Hostnames and interface addressing

Each router is configured in global configuration mode, then each interface is configured in interface mode. R1 is shown here as an example. R2, R3 and R4 follow the same pattern with the addresses in the Addressing Table.

```
enable
configure terminal
hostname R1
no ip domain-lookup

interface GigabitEthernet0/0
 description LAN A1 - SW1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit

interface Serial0/0/0
 description WAN to R2
 ip address 172.16.0.1 255.255.255.252
 no shutdown
 exit
end
write memory
```

| Command | Mode | What it does |
|---|---|---|
| `enable` | User EXEC | Enters privileged EXEC mode. |
| `configure terminal` | Privileged EXEC | Enters global configuration mode. |
| `hostname R1` | Global config | Sets the device name shown in the prompt. |
| `no ip domain-lookup` | Global config | Stops the router from trying to resolve mistyped commands as hostnames, which avoids long delays. |
| `interface GigabitEthernet0/0` | Global config | Opens the configuration for that interface. |
| `ip address 192.168.10.1 255.255.255.0` | Interface config | Assigns the IPv4 address and mask. |
| `no shutdown` | Interface config | Turns the interface on. Router interfaces start administratively down by default. |
| `description` | Interface config | Adds a note that documents the link. It has no effect on forwarding. |
| `end` | Any config mode | Returns to privileged EXEC mode. |
| `write memory` | Privileged EXEC | Saves the running configuration to startup configuration in NVRAM. |

**Notes:** Interfaces start shut down on routers, so `no shutdown` is required on every interface you use. The `description` line is optional, but it makes `show running-config` easier to read. Changes made in configuration mode live only in RAM until `write memory` is run.

### Step 2: Static routes

Each router gets one static route for every remote network. Next hops are the neighbouring router's address on the shared WAN link.

| Router | Static route | Next hop | Purpose |
|---|---|---|---|
| R1 | 192.168.20.0/24, 192.168.30.0/24, 192.168.40.0/24, 192.168.50.0/24, 192.168.60.0/24 | 172.16.0.2 (R2) | Reaches all LANs behind R2, R4 and R3 |
| R1 | 172.16.0.4/30, 172.16.0.8/30 | 172.16.0.2 (R2) | Reaches the R2 to R4 and R4 to R3 links |
| R2 | 192.168.10.0/24 | 172.16.0.1 (R1) | Return route to LAN A1 |
| R2 | 192.168.40.0/24, 192.168.50.0/24, 192.168.60.0/24 | 172.16.0.6 (R4) | Reaches Building B through R4 |
| R2 | 172.16.0.8/30 | 172.16.0.6 (R4) | Reaches the R4 to R3 link |
| R3 | 192.168.10.0/24, 192.168.20.0/24, 192.168.30.0/24 | 172.16.0.9 (R4) | Return route to Building A through R4 |
| R3 | 192.168.50.0/24, 192.168.60.0/24 | 172.16.0.9 (R4) | Reaches LANs B2 and B3 through R4 |
| R3 | 172.16.0.0/30, 172.16.0.4/30 | 172.16.0.9 (R4) | Reaches the R1 to R2 and R2 to R4 links |
| R4 | 192.168.10.0/24, 192.168.20.0/24, 192.168.30.0/24 | 172.16.0.5 (R2) | Reaches Building A through R2 |
| R4 | 172.16.0.0/30 | 172.16.0.5 (R2) | Reaches the R1 to R2 link |
| R4 | 192.168.40.0/24 | 172.16.0.10 (R3) | Reaches LAN B1 through R3 |

For R1, the configuration commands are:

```
configure terminal
ip route 192.168.20.0 255.255.255.0 172.16.0.2
ip route 192.168.30.0 255.255.255.0 172.16.0.2
ip route 192.168.40.0 255.255.255.0 172.16.0.2
ip route 192.168.50.0 255.255.255.0 172.16.0.2
ip route 192.168.60.0 255.255.255.0 172.16.0.2
ip route 172.16.0.4 255.255.255.252 172.16.0.2
ip route 172.16.0.8 255.255.255.252 172.16.0.2
end
write memory
```

The other three routers use the same syntax with the routes listed in the table above:

| Command | Mode | What it does |
|---|---|---|
| `ip route <network> <mask> <next-hop>` | Global config | Adds a static route. The router forwards traffic for that network to the next-hop address. |
| `end` | Any config mode | Returns to privileged EXEC mode. |
| `write memory` | Privileged EXEC | Saves the configuration so the routes survive a reload. |

**Notes:** Each route uses a next-hop address, which must be on a directly connected network. The next hop for R2 to R3 traffic is R4, not R3, because R2 and R3 do not share a link. A missing return route is the most common cause of failed pings in this topology. Confirm that each router has an entry for every LAN and every WAN link.

## ✅ Verification & Results

| Check | Command / Action | What success looks like | Result |
|---|---|---|---|
| Interfaces are up | `show ip interface brief` on each router | Every interface in use shows `up` and `up` | Expected (not captured) |
| Routing table is complete | `show ip route` on each router | Connected (`C`) entries for local networks and static (`S`) entries for every remote network | Expected (not captured) |
| Serial links are up | `show ip interface brief` on R1, R2, R3 and R4 | Serial interfaces show `up` and `up` | Expected (not captured) |
| Router to router ping | `ping 172.16.0.6` on R2 | Five `!` characters | Expected (not captured) |
| Cross-building ping | `ping 192.168.60.2` from a PC in LAN A1 | Replies received | Expected (not captured) |
| Path check | `tracert 192.168.40.2` from a PC in LAN A1 | Hops through 192.168.10.1, 172.16.0.2, 172.16.0.6, 172.16.0.10, then the destination | Expected (not captured) |

**Expected output** of `show ip route` on R1, showing the static routes. Labelled as expected, not captured:

```
Gateway of last resort is not set

C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
S    192.168.20.0/24 [1/0] via 172.16.0.2
S    192.168.30.0/24 [1/0] via 172.16.0.2
S    192.168.40.0/24 [1/0] via 172.16.0.2
S    192.168.50.0/24 [1/0] via 172.16.0.2
S    192.168.60.0/24 [1/0] via 172.16.0.2
C    172.16.0.0/30 is directly connected, Serial0/0/0
S    172.16.0.4/30 [1/0] via 172.16.0.2
S    172.16.0.8/30 [1/0] via 172.16.0.2
```

**Expected output** of `tracert` from a PC in LAN A1 to a PC in LAN B1. Each line is one router hop:

```
1    192.168.10.1
2    172.16.0.2
3    172.16.0.6
4    172.16.0.10
5    192.168.40.2
```

Each router hop in the trace matches the routing decisions in the Step 2 table. If a hop is missing or times out, check the routing table on the router before that hop.

## 🛠️ Troubleshooting Notes

The table below lists the most likely failures for a static routing lab with serial links. These are general troubleshooting tips and not problems that were encountered in this lab.

| Symptom | Likely cause | How to check | Fix |
|---|---|---|---|
| Interface shows administratively down | `no shutdown` was not entered | `show ip interface brief` | Enter `no shutdown` on that interface. |
| Serial interface is up/down | Missing `clock rate` on the DCE end | `show controllers serial 0/0/0` | Add `clock rate 64000` on the DCE end only. |
| Ping to a remote LAN fails, but the local gateway responds | Missing static route on the router that sends the packet | `show ip route` | Add the missing `ip route` command. |
| Ping goes out but replies never return | Missing return route on a router in the reply path | `show ip route` on each router in the path | Add a route back to the source network. |
| Ping to a remote LAN fails, but the first ping times out | ARP resolution, which is normal behaviour | Repeat the ping | No fix needed. Later pings succeed. |
| Pings fail from one building only | Wrong PC default gateway | `ipconfig /all` on the PC | Set the default gateway to the LAN's .1 address. |
| Route exists but points to the wrong next hop | Typo in the next-hop IP | `show running-config` compared with the Addressing Table | Remove the route with `no ip route ...` and enter it again. |

## 💼 Skills Demonstrated

- Configured IPv4 addresses on 12 router interfaces across four routers, and verified each interface with `show ip interface brief`.
- Calculated subnet boundaries, broadcast addresses and usable host ranges for six /24 LANs and three /30 WAN links.
- Designed static routes in both directions so that traffic between every pair of LANs has a return path.
- Chose next-hop static routes over exit-interface routes for point-to-point serial links.
- Planned the topology from a diagram and documented the address plan and cabling in a reproducible table.

**CCNA 200-301 mapping:** 3.0 IP Connectivity (static routing, routing table, administrative distance), 1.0 Network Fundamentals (IPv4 addressing and subnetting, cable types, interface troubleshooting).

## 📂 Files in This Lab

```
Lab-N-Static-Routing/
├── README.md
├── Capture.JPG               # Topology diagram used in this README
└── Static-Routing.pkt        # Packet Tracer file for the lab
```

## 🚀 How to Open This Lab

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco NetAcad account).
2. Open `Static-Routing.pkt`.
3. Click any router, open the CLI tab, and run `show ip route` to see its static routes. Run `show ip interface brief` to check interface status. From a PC, open Desktop, then Command Prompt, and ping a PC in another building.

## 🔭 Possible Improvements

- Replace static routes with OSPF so the network can recover automatically when a serial link fails, and compare the routing table and configuration length with the static version.
- Add a floating static route through a backup path, with an administrative distance of 200, so traffic switches over only when the primary link is down.
- Restrict access to the routers with SSH and a local username and password, and remove the console and enable passwords from plain text.
- Add a standard or extended ACL on R2 to permit traffic from LAN A1 to LAN B1 only, and verify it with `show access-lists`.

## 👤 Author

Anish Raj — Aspiring Network Engineer, building CCNA / networking fundamentals skills.

---
*This lab is part of my ongoing CCNA / Network Administrator study roadmap.*
