[← Back to all labs](../README.md)

# Default Routing: Connecting Three Networks with Default Routes (Cisco Packet Tracer)

Three routers connected in a line by serial links, each with its own LAN. Instead of building a full routing table on every router, default routes (`0.0.0.0/0`) are used to forward traffic for unknown networks toward the next router.

## 📋 At a Glance

| | |
|---|---|
| **Lab number** | 3 |
| **Level** | Beginner, CCNA |
| **Tools** | Cisco Packet Tracer, Cisco IOS CLI |
| **Devices** | 3 routers (Cisco 2811 series), 3 switches (2960-24TT), 6 PCs |
| **Protocols / features** | Default routing (static default route), serial WAN links, connected routes |
| **Networks** | 10.0.0.0/8, 11.0.0.0/8, 12.0.0.0/8, 13.0.0.0/8, 20.0.0.0/8 |
| **CCNA domains** | IP Connectivity, Network Fundamentals |

## 📌 Project Objective

Build a three-router network with three LANs (11.0.0.0/8, 12.0.0.0/8, 13.0.0.0/8) joined by two serial links (10.0.0.0/8 and 20.0.0.0/8), and configure default routes so that traffic for networks a router does not know about is sent to a neighbouring router. Success is shown when `show ip route` on each router lists a static default route (`S*`) and a gateway of last resort.

## 🌍 Real-World Scenario

A small branch office usually has one link to the rest of the network. Its router does not need to know every network in the company or on the internet, so it carries a single default route pointing at the upstream router (the ISP or head office). The same idea is used on almost every internet edge router. This lab reproduces that pattern on a small scale: R1 and R3 are the "branch" routers, and R2 sits in the middle.

## 🖧 Topology

![Default routing network topology with three routers, three switches and six PCs](Default_Routing_Topology.JPG)

R1 (green LAN, 11.0.0.0/8) connects to R2 (blue LAN, 12.0.0.0/8) over a serial link in 10.0.0.0/8. R2 connects to R3 (yellow LAN, 13.0.0.0/8) over a second serial link in 20.0.0.0/8. Each router has a 2960 switch on its FastEthernet port with two PCs attached. The clock icons on the topology mark the DCE ends of the serial cables (R1 Se0/0/0 and R2 Se0/0/0).

### Cabling

| From | Port | To | Port | Cable type |
|---|---|---|---|---|
| R1 | Se0/0/0 (DCE) | R2 | Se0/0/1 (DTE) | Serial DCE/DTE |
| R2 | Se0/0/0 (DCE) | R3 | Se0/0/1 (DTE) | Serial DCE/DTE |
| R1 | Fa0/0 | Switch (R1 LAN) | Fa0/1 | Straight-through copper* |
| Switch (R1 LAN) | Fa0/2 | PC 11.0.0.3 | Fa0 | Straight-through copper* |
| Switch (R1 LAN) | Fa0/3 | PC 11.0.0.2 | Fa0 | Straight-through copper* |
| R2 | Fa0/0 | Switch (R2 LAN) | Fa0/1 | Straight-through copper* |
| Switch (R2 LAN) | Fa0/2 | PC 12.0.0.2 | Fa0 | Straight-through copper* |
| Switch (R2 LAN) | Fa0/3 | PC 12.0.0.3 | Fa0 | Straight-through copper* |
| R3 | Fa0/0 | Switch (R3 LAN) | Fa0/1 | Straight-through copper* |
| Switch (R3 LAN) | Fa0/2 | PC 13.0.0.2 | Fa0 | Straight-through copper* |
| Switch (R3 LAN) | Fa0/3 | PC 13.0.0.3 | Fa0 | Straight-through copper* |

\*Copper cable type is not labelled in the topology image. Straight-through is the standard choice for router-to-switch and switch-to-PC links (different device layers). The serial DCE end is taken from the clock icons in the topology.

### Addressing Table

Router addresses come from the topology image and the `show ip route` screenshots (they agree). All masks are 255.0.0.0 (/8), as shown by the `/8` connected routes in the screenshots.

| Device | Interface | IP Address | Subnet Mask | Prefix | Default Gateway |
|---|---|---|---|---|---|
| R1 | Se0/0/0 | 10.0.0.1 | 255.0.0.0 | /8 | n/a |
| R1 | Fa0/0 | 11.0.0.1 | 255.0.0.0 | /8 | n/a |
| R2 | Se0/0/1 | 10.0.0.2 | 255.0.0.0 | /8 | n/a |
| R2 | Se0/0/0 | 20.0.0.1 | 255.0.0.0 | /8 | n/a |
| R2 | Fa0/0 | 12.0.0.1 | 255.0.0.0 | /8 | n/a |
| R3 | Se0/0/1 | 20.0.0.2 | 255.0.0.0 | /8 | n/a |
| R3 | Fa0/0 | 13.0.0.1 | 255.0.0.0 | /8 | n/a |
| PC | Fa0 | 11.0.0.2 | 255.0.0.0 (assumed) | /8 | 11.0.0.1 (assumed) |
| PC | Fa0 | 11.0.0.3 | 255.0.0.0 (assumed) | /8 | 11.0.0.1 (assumed) |
| PC | Fa0 | 12.0.0.2 | 255.0.0.0 (assumed) | /8 | 12.0.0.1 (assumed) |
| PC | Fa0 | 12.0.0.3 | 255.0.0.0 (assumed) | /8 | 12.0.0.1 (assumed) |
| PC | Fa0 | 13.0.0.2 | 255.0.0.0 (assumed) | /8 | 13.0.0.1 (assumed) |
| PC | Fa0 | 13.0.0.3 | 255.0.0.0 (assumed) | /8 | 13.0.0.1 (assumed) |

PC masks and gateways are not visible in the topology image. They are shown as "assumed" because each PC must use the mask of its LAN and the router's Fa0/0 address as its gateway for this lab to work.

### Subnet Analysis

| Network | Mask | Network Address | First Usable | Last Usable | Broadcast | Usable Hosts | Private (RFC 1918)? |
|---|---|---|---|---|---|---|---|
| 10.0.0.0/8 (R1–R2 link) | 255.0.0.0 | 10.0.0.0 | 10.0.0.1 | 10.255.255.254 | 10.255.255.255 | 16,777,214 | Yes |
| 11.0.0.0/8 (R1 LAN) | 255.0.0.0 | 11.0.0.0 | 11.0.0.1 | 11.255.255.254 | 11.255.255.255 | 16,777,214 | No (public range) |
| 12.0.0.0/8 (R2 LAN) | 255.0.0.0 | 12.0.0.0 | 12.0.0.1 | 12.255.255.254 | 12.255.255.255 | 16,777,214 | No (public range) |
| 13.0.0.0/8 (R3 LAN) | 255.0.0.0 | 13.0.0.0 | 13.0.0.1 | 13.255.255.254 | 13.255.255.255 | 16,777,214 | No (public range) |
| 20.0.0.0/8 (R2–R3 link) | 255.0.0.0 | 20.0.0.0 | 20.0.0.1 | 20.255.255.254 | 20.255.255.255 | 16,777,214 | No (public range) |

- Each LAN router interface uses the first address of its network (.1), and the PCs use .2 and .3, so every host and gateway sits inside its own subnet.
- Only 10.0.0.0/8 is private. The 11, 12, 13 and 20 ranges belong to the public internet; they are fine inside a Packet Tracer simulation but should never be used on a real private network.
- A point-to-point serial link only needs two addresses. A /30 would be the efficient choice in production; /8 is used here for simplicity.

## 📚 Concepts Behind This Lab

**Default route.** A route to `0.0.0.0/0`. The prefix length is 0, so it matches every destination, but because routers use the *longest prefix match*, it is only used when nothing more specific exists in the routing table. It is shown in the table as `S*` (static, candidate default).

**Gateway of last resort.** The next-hop address of the default route. `show ip route` prints it at the top: `Gateway of last resort is 10.0.0.2 to network 0.0.0.0`. If it says "not set", there is no default route.

**Stub router.** A router with only one connection to the rest of the network. R1 and R3 are stubs: all their outside traffic must go through R2. A default route is the ideal fit because there is only one possible next hop.

**Connected (`C`) and local (`L`) routes.** When an interface has an IP address and is up/up, the router adds a `C` route for the network and an `L` route (/32) for its own address. These are visible in all three screenshots. Directly connected networks need no static or default route.

**Administrative distance (AD).** Trust level of a route source. Connected = 0, static = 1. The `[1/0]` in `S* 0.0.0.0/0 [1/0]` means AD 1, metric 0.

**Routing works in both directions.** A ping needs a route to the destination and a route back for the reply. This is why all three routers need a default route (or specific routes) pointing the right way.

| | Default route | Static route to a specific network |
|---|---|---|
| Syntax | `ip route 0.0.0.0 0.0.0.0 <next-hop>` | `ip route 13.0.0.0 255.0.0.0 <next-hop>` |
| Matches | Anything with no better match | Only that network |
| Best for | Stub routers, internet edge | Transit routers that need precise paths |
| Table size | Very small | Grows with every network added |

## 🧭 Lab Steps

1. Place three routers, three switches and six PCs, and cable them as in the topology.
2. Assign IP addresses to the serial and FastEthernet interfaces of each router and bring the interfaces up.
3. Configure the DCE side of each serial link with a clock rate.
4. Configure IP addresses on the PCs.
5. Configure a default route on R1 (via R2), on R3 (via R2), and on R2 (see the note in Step 3 below).
6. Check `show ip route` on each router to confirm the `S*` default route and the gateway of last resort.

The steps are put together from the topology and the `show ip route` screenshots; the author's own step list was not supplied.

## ⚙️ Configuration

> The screenshots show the routing tables, not the typed `ip address` or `ip route` commands. The commands below are **reconstructed from the routing table output and the topology**, so they match what the screenshots prove. The clock rate value is not shown anywhere, so it is left as a placeholder.

### Step 1: Configure router interfaces

```
! R1
configure terminal
interface serial 0/0/0
 ip address 10.0.0.1 255.0.0.0
 clock rate <value>
 no shutdown
interface fastEthernet 0/0
 ip address 11.0.0.1 255.0.0.0
 no shutdown

! R2
configure terminal
interface serial 0/0/1
 ip address 10.0.0.2 255.0.0.0
 no shutdown
interface serial 0/0/0
 ip address 20.0.0.1 255.0.0.0
 clock rate <value>
 no shutdown
interface fastEthernet 0/0
 ip address 12.0.0.1 255.0.0.0
 no shutdown

! R3
configure terminal
interface serial 0/0/1
 ip address 20.0.0.2 255.0.0.0
 no shutdown
interface fastEthernet 0/0
 ip address 13.0.0.1 255.0.0.0
 no shutdown
```

| Command | Mode | What it does |
|---|---|---|
| `configure terminal` | Privileged EXEC | Enters global configuration mode (prompt `Router(config)#`). |
| `interface serial 0/0/0` | Global config | Selects the interface; prompt becomes `Router(config-if)#`. |
| `ip address A.B.C.D mask` | Interface config | Sets the IPv4 address and mask. |
| `clock rate <value>` | Interface config | Supplies the clock signal. Only the **DCE** end needs it (R1 Se0/0/0 and R2 Se0/0/0). |
| `no shutdown` | Interface config | Brings the interface up. Router interfaces are administratively down by default. |

**Notes:** The serial link only comes up when the DCE side has a clock rate and both ends have `no shutdown`. The three screenshots show every one of these interfaces as `C` (connected) with an `L` entry, which proves they were addressed and up.

### Step 2: Check the connected networks

![R1 show ip route output with default route via 10.0.0.2](R1_IP_route.JPG)

Visible in the screenshot (typed at the CLI):

```
Router#
Router#configure terminal
Router(config)#do show ip route
```

| Command | Mode | What it does |
|---|---|---|
| `do show ip route` | Global config | Runs the privileged EXEC command `show ip route` without leaving config mode. |

**Notes:** The prompt in all three screenshots is `Router`, so the hostnames R1, R2 and R3 exist only as labels in the topology; no `hostname` command was applied.

### Step 3: Configure the default routes

```
! R1: send everything unknown to R2
ip route 0.0.0.0 0.0.0.0 10.0.0.2

! R3: send everything unknown to R2
ip route 0.0.0.0 0.0.0.0 20.0.0.1

! R2: two default routes, as shown in its routing table
ip route 0.0.0.0 0.0.0.0 20.0.0.2
ip route 0.0.0.0 0.0.0.0 10.0.0.1
```

| Command | Mode | What it does |
|---|---|---|
| `ip route 0.0.0.0 0.0.0.0 <next-hop>` | Global config | Creates a static default route. `0.0.0.0 0.0.0.0` means "every network". The next hop must be an address on a directly connected network. |

**Notes:**
- The next hop is always the neighbour's address on the shared link: R1 uses 10.0.0.2 (R2), R3 uses 20.0.0.1 (R2), and R2 uses 20.0.0.2 (R3) and 10.0.0.1 (R1).
- R2 is not a stub router: it has two neighbours. Two equal default routes make R2 split traffic between R1 and R3 rather than choosing the right one for each destination. See the Troubleshooting section for why this matters.

## ✅ Verification & Results

![R2 show ip route output with two default routes](R2_IP_route.JPG)

![R3 show ip route output with default route via 20.0.0.1](R3_IP_route.JPG)

| Check | Command / Action | What success looks like | Result |
|---|---|---|---|
| R1 default route | `show ip route` on R1 | `S* 0.0.0.0/0 [1/0] via 10.0.0.2` and gateway of last resort 10.0.0.2 | Shown in screenshot |
| R2 default routes | `show ip route` on R2 | `S* 0.0.0.0/0` via 20.0.0.2 and via 10.0.0.1; gateway of last resort 20.0.0.2 | Shown in screenshot |
| R3 default route | `show ip route` on R3 | `S* 0.0.0.0/0 [1/0] via 20.0.0.1` and gateway of last resort 20.0.0.1 | Shown in screenshot |
| Connected networks | `show ip route` on each router | `C` and `L` entries for every addressed interface | Shown in screenshot |
| Interfaces up | `show ip interface brief` | Status up, Protocol up on every used interface | Not captured |
| End-to-end ping | `ping` between PCs on different LANs | Replies from the far PC | Not captured |

### Captured output: R1

```
Gateway of last resort is 10.0.0.2 to network 0.0.0.0

     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       10.0.0.0/8 is directly connected, Serial0/0/0
L       10.0.0.1/32 is directly connected, Serial0/0/0
     11.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       11.0.0.0/8 is directly connected, FastEthernet0/0
L       11.0.0.1/32 is directly connected, FastEthernet0/0
S*   0.0.0.0/0 [1/0] via 10.0.0.2
```

### Captured output: R2

```
Gateway of last resort is 20.0.0.2 to network 0.0.0.0

     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       10.0.0.0/8 is directly connected, Serial0/0/1
L       10.0.0.2/32 is directly connected, Serial0/0/1
     12.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       12.0.0.0/8 is directly connected, FastEthernet0/0
L       12.0.0.1/32 is directly connected, FastEthernet0/0
     20.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       20.0.0.0/8 is directly connected, Serial0/0/0
L       20.0.0.1/32 is directly connected, Serial0/0/0
S*   0.0.0.0/0 [1/0] via 20.0.0.2
             [1/0] via 10.0.0.1
```

### Captured output: R3

```
Gateway of last resort is 20.0.0.1 to network 0.0.0.0

     13.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       13.0.0.0/8 is directly connected, FastEthernet0/0
L       13.0.0.1/32 is directly connected, FastEthernet0/0
     20.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       20.0.0.0/8 is directly connected, Serial0/0/1
L       20.0.0.2/32 is directly connected, Serial0/0/1
S*   0.0.0.0/0 [1/0] via 20.0.0.1
```

**Reading the output:** "variably subnetted, 2 subnets, 2 masks" is normal. It counts the /8 network and the /32 local route for each address. It does not mean the lab uses VLSM.

### Commands to test connectivity (not captured)

```
! On a PC (Command Prompt tab)
ping 11.0.0.1
ping 13.0.0.2
tracert 13.0.0.2

! On a router
ping 20.0.0.2
```

**Expected output (general example, not captured from this lab):** a router ping that works prints `!!!!!` and `Success rate is 100 percent (5/5)`. The first ping from a PC may time out once while ARP resolves; this is normal.

## 🛠️ Troubleshooting Notes

No problems were reported by the author. The table below lists general failure causes for this type of lab.

| Symptom | Likely cause | How to check | Fix |
|---|---|---|---|
| Interface shows administratively down | Missing `no shutdown` | `show ip interface brief` | Enter `no shutdown` on the interface |
| Serial interface up/down | No clock rate on the DCE end, or mismatched encapsulation | `show controllers serial 0/0/0`, `show interfaces serial 0/0/0` | Add `clock rate` on the DCE side |
| Gateway of last resort is not set | Default route missing or next hop not reachable | `show ip route`, `show running-config \| include ip route` | Re-enter `ip route 0.0.0.0 0.0.0.0 <next-hop>` with a next hop on a connected network |
| Ping to the gateway works, ping to another LAN fails | Missing return route, or wrong PC gateway | `ipconfig` on the PC, `show ip route` on each router | Fix the gateway; make sure every router can route toward the source |
| "TTL expired in transit" or intermittent replies | Routing loop between routers | `tracert` shows the same two routers repeating | Remove the conflicting route (see below) |

**Design point to be aware of (general analysis, not observed in this lab):** R2 has a default route toward R3 *and* one toward R1. A packet for 13.0.0.0/8 should go to R3, but R2 may also send it back to R1. R1 would then return it to R2 using its own default route, and the packet would loop until its TTL runs out. Because R2 is a transit router with two neighbours, the safer design is two specific static routes: `ip route 11.0.0.0 255.0.0.0 10.0.0.1` and `ip route 13.0.0.0 255.0.0.0 20.0.0.2`. Default routes belong on the stub routers (R1 and R3).

## 💼 Skills Demonstrated

- Designed and addressed a multi-router topology with LAN and WAN segments.
- Configured serial WAN links, including identifying the DCE side that needs a clock rate.
- Configured static default routes and identified the gateway of last resort.
- Read and interpreted the routing table (`C`, `L`, `S*` codes, administrative distance and metric).
- Explained when to use a default route (stub routers) and when more specific routes are needed (transit routers).
- Verified router configuration with `show ip route`.

**CCNA 200-301 mapping:** 3.0 IP Connectivity (interpret the components of a routing table; configure and verify IPv4 static routing, including default routes); 1.0 Network Fundamentals (describe routing concepts, IPv4 addressing, interface and cable troubleshooting).

## 📂 Files in This Lab

```
Default_Routing/
├── README.md
├── Default_Routing.pkt              # Packet Tracer lab file
├── Default_Routing_Topology.JPG     # Network topology
├── R1_IP_route.JPG                  # show ip route on R1
├── R2_IP_route.JPG                  # show ip route on R2
└── R3_IP_route.JPG                  # show ip route on R3
```

## 🚀 How to Open This Lab

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco NetAcad account).
2. Open `Default_Routing.pkt`.
3. Click a router, open the **CLI** tab, press Enter, and run `enable` then `show ip route` to see the default route.
4. Click a PC, open **Desktop > Command Prompt**, and run `ping` or `tracert` to another LAN.

## 🔭 Possible Improvements

- Replace the two default routes on R2 with specific static routes to 11.0.0.0/8 and 13.0.0.0/8 to avoid any chance of a loop.
- Set hostnames (R1, R2, R3) so the CLI prompts match the topology.
- Use /30 subnets on the serial links and private (RFC 1918) address ranges on the LANs.
- Replace the static routes with a dynamic protocol (RIP, EIGRP or OSPF) and compare the routing tables.

## 👤 Author

[Your Name] — Aspiring Network Engineer, building CCNA / networking fundamentals skills.

---
*This lab is part of my ongoing CCNA / Network Administrator study roadmap.*
