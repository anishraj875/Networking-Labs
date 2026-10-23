[← Back to all labs](../README.md)

# EIGRP Routing Across Three Branch Routers with TFTP Config Backup (Cisco Packet Tracer)

Three routers (Accounts, HR and Sales) are connected in a chain over serial links and configured with EIGRP (AS 100) so every LAN can learn routes to every other LAN. Each router's configuration is then backed up to a TFTP server and saved to NVRAM.

## 📋 At a Glance

| | |
|---|---|
| **Lab file** | `EIGRP_Lab.pkt` |
| **Level** | Intermediate, CCNA |
| **Tools** | Cisco Packet Tracer, Cisco IOS CLI |
| **Devices** | 3 routers (Cisco 2811), 3 switches (2960-24), 6 PCs, 1 server (Server-PT) |
| **Protocols / features** | EIGRP (AS 100), serial point-to-point links (DCE/DTE), `enable secret`, TFTP backup |
| **Networks** | 192.168.10.0/24, 192.168.11.0/24, 192.168.12.0/24 (LANs); 10.0.0.0/8, 11.0.0.0/8 (serial links) |
| **CCNA domains** | IP Connectivity, Network Fundamentals, IP Services, Security Fundamentals |

## 📌 Project Objective

Build a three-site network (Accounts, HR, Sales), give every router a hostname and an `enable secret`, address all interfaces, and enable EIGRP in autonomous system 100 on every router so the routers form neighbor adjacencies and exchange routes dynamically. Finally, back up each router's running configuration to a TFTP server (192.168.12.4) and save it to startup configuration.

Success looks like: EIGRP neighbor adjacencies come up, remote LANs appear as EIGRP routes, and each backup copies to the TFTP server successfully.

## 🌍 Real-World Scenario

A company with several departments or branch offices usually connects its sites through routers. Maintaining static routes by hand on every router does not scale, so a dynamic routing protocol such as EIGRP is used: routers discover each other, share the networks they know about, and adapt when a link fails. Regular configuration backups to a central server, as done here with TFTP, let an administrator restore a failed router quickly. (This lab is a practice environment, not a production deployment.)

## 🖧 Topology

![Network topology showing Accounts, HR and Sales routers in a chain with switches, PCs and a server on the Sales LAN](Topology.JPG)

The Accounts router (Router0) connects to the HR router (Router1), which connects to the Sales router (Router2) through two serial links. Each router has a LAN on its FastEthernet 0/0 interface, connected to a 2960-24 switch that serves two PCs. The Sales switch also connects the Server-PT (192.168.12.4), which is used as the TFTP server. The topology image labels each site with its enable password: `Accounts@1`, `HR@1` and `Sales@1`.

### Cabling

| From | Port | To | Port | Cable type |
|---|---|---|---|---|
| Accounts router | Se0/0/0 (DCE) | HR router | Se0/0/1 | Serial DCE |
| HR router | Se0/0/0 (DCE) | Sales router | Se0/0/1 | Serial DCE |
| Accounts router | Fa0/0 | Accounts switch | Gig0/1 | Copper straight-through |
| Accounts switch | Fa0/1 | Accounts PC (left) | Fa0 | Copper straight-through |
| Accounts switch | Fa0/2 | Accounts PC (right) | Fa0 | Copper straight-through |
| HR router | Fa0/0 | HR switch | Gig0/1 | Copper straight-through |
| HR switch | Fa0/1 | HR PC (left) | Fa0 | Copper straight-through |
| HR switch | Fa0/2 | HR PC (right) | Fa0 | Copper straight-through |
| Sales router | Fa0/0 | Sales switch | Gig0/1 | Copper straight-through |
| Sales switch | Fa0/1 | Sales PC (left) | Fa0 | Copper straight-through |
| Sales switch | Fa0/2 | Sales PC (right) | Fa0 | Copper straight-through |
| Sales switch | Fa0/3 | Server-PT | Fa0 | Copper straight-through |

The router-to-switch and switch-to-end-device links follow the standard rule (unlike devices use straight-through). On each serial link, the end with the `clock rate` command is the DCE end and supplies the clock.

### Addressing Table

| Device | Interface | IP Address | Subnet Mask | Prefix | Gateway label shown in topology |
|---|---|---|---|---|---|
| Accounts | Se0/0/0 (DCE) | 10.0.0.1 | 255.0.0.0 | /8 | N/A |
| Accounts | Fa0/0 | 192.168.10.1 | 255.255.255.0 | /24 | N/A |
| HR | Se0/0/1 | 10.0.0.2 | 255.0.0.0 | /8 | N/A |
| HR | Se0/0/0 (DCE) | 11.0.0.1 | 255.0.0.0 | /8 | N/A |
| HR | Fa0/0 | 192.168.11.1 | 255.255.255.0 | /24 | N/A |
| Sales | Se0/0/1 | 11.0.0.2 | 255.0.0.0 | /8 | N/A |
| Sales | Fa0/0 | 192.168.12.1 | 255.255.255.0 | /24 | N/A |
| Accounts PC (left) | Fa0 | 192.168.10.2 | 255.255.255.0 | /24 | 10.0.0.1 |
| Accounts PC (right) | Fa0 | 192.168.10.3 | 255.255.255.0 | /24 | 10.0.0.1 |
| HR PC (left) | Fa0 | 192.168.11.2 | 255.255.255.0 | /24 | 10.0.0.2 |
| HR PC (right) | Fa0 | 192.168.11.3 | 255.255.255.0 | /24 | 11.0.0.1 |
| Sales PC (left) | Fa0 | 192.168.12.2 | 255.255.255.0 | /24 | 11.0.0.2 |
| Sales PC (right) | Fa0 | 192.168.12.3 | 255.255.255.0 | /24 | 10.0.0.2 |
| TFTP Server | Fa0 | 192.168.12.4 | 255.255.255.0 | /24 | 11.0.0.2 |

> **Note on the gateway labels.** The last column is copied exactly as written in the topology image. Those labels name router serial addresses (10.0.0.x / 11.0.0.x), which are not inside the hosts' own 192.168.x.0/24 subnets. A host's default gateway must be an address on its own subnet, so the expected gateway for each LAN is the router's Fa0/0 address (192.168.10.1, 192.168.11.1 and 192.168.12.1). Check each device's IP Configuration in the `.pkt` file to confirm what is actually set.

**Device access (lab only):** `enable secret` values shown in the topology image and CLI screenshots are `Accounts@1`, `HR@1` and `Sales@1`. These are practice passwords; real networks use unique, strong credentials.

### Subnet Analysis

| Network | Mask | Network Address | First Usable | Last Usable | Broadcast | Usable Hosts | Private (RFC 1918)? |
|---|---|---|---|---|---|---|---|
| Accounts LAN | /24 | 192.168.10.0 | 192.168.10.1 | 192.168.10.254 | 192.168.10.255 | 254 | Yes |
| HR LAN | /24 | 192.168.11.0 | 192.168.11.1 | 192.168.11.254 | 192.168.11.255 | 254 | Yes |
| Sales LAN | /24 | 192.168.12.0 | 192.168.12.1 | 192.168.12.254 | 192.168.12.255 | 254 | Yes |
| Accounts–HR link | /8 | 10.0.0.0 | 10.0.0.1 | 10.255.255.254 | 10.255.255.255 | 16,777,214 | Yes |
| HR–Sales link | /8 | 11.0.0.0 | 11.0.0.1 | 11.255.255.254 | 11.255.255.255 | 16,777,214 | **No (public range)** |

- Each router uses `.1` on its LAN as the router interface, with hosts at `.2`, `.3` and the server at `.4` on the Sales LAN.
- The two serial links use the full Class A mask (/8) with only two devices on each. A /30 (2 usable hosts) would be the efficient choice for point-to-point links, but /8 works for the lab.
- 11.0.0.0/8 is not an RFC 1918 private range; it is publicly routable address space. It is fine inside an isolated simulation, but should not be used for internal links on a real network.

## 📚 Concepts Behind This Lab

**EIGRP (Enhanced Interior Gateway Routing Protocol).** An advanced distance-vector protocol from Cisco that uses the DUAL algorithm to select loop-free paths and converge quickly. Routers discover neighbors with Hello packets (multicast 224.0.0.10), exchange routes in partial, triggered updates (not periodic full-table dumps), and keep three tables: neighbor, topology and routing. Internal EIGRP routes have an administrative distance of 90, shown as `D` in `show ip route`.

**Autonomous system number.** The number in `router eigrp 100` must be the same on routers that want to become neighbors. All three routers here use AS 100.

**The `network` statement.** `network` tells EIGRP two things: which interfaces to run on (any interface whose IP falls in that network), and which networks to advertise. When entered without a wildcard mask, as in this lab, it matches the whole classful network (for example `10.0.0.0` means 10.0.0.0/8, `192.168.10.0` means /24).

**Default metric.** EIGRP combines bandwidth (slowest link in the path) and cumulative delay. Note that the `clock rate 64000` command only sets the physical clocking speed of the serial line; EIGRP uses the interface's configured `bandwidth` value, not the clock rate.

**Serial links: DCE and DTE.** In a back-to-back serial connection, the DCE end provides the clock. The `clock rate` command is entered only on the DCE end (Accounts Se0/0/0 and HR Se0/0/0 here). Without it, the line protocol will not come up.

**`enable secret` vs `enable password`.**

| | `enable password` | `enable secret` |
|---|---|---|
| Stored as | Plain text (unless `service password-encryption`) | Hash |
| Strength | Weak | Stronger |
| If both exist | Ignored | Wins |

**TFTP.** Trivial File Transfer Protocol (UDP port 69) copies files with no authentication or encryption, which makes it suitable for labs and trusted networks only. `copy running-config tftp` sends the active configuration to the server; the default filename offered is `<hostname>-confg`, which the author overrode with names like `Accounts_Config`.

**Running vs startup configuration.** Changes made at the CLI live in RAM (running-config) and are lost on reload until saved to NVRAM (startup-config) with `write memory` or `copy running-config startup-config`.

## 🧭 Lab Steps

1. Build the topology: three routers, three 2960 switches, six PCs and one server; connect serial links between routers and straight-through links to the LANs.
2. Configure the Accounts router: hostname, `enable secret`, Se0/0/0 (with clock rate) and Fa0/0.
3. Configure the HR router: hostname, `enable secret`, Se0/0/1, Se0/0/0 (with clock rate) and Fa0/0.
4. Configure the Sales router: hostname, `enable secret`, Se0/0/1 and Fa0/0.
5. Check the routing table on each router (connected routes only at this stage).
6. Enable EIGRP AS 100 on each router and advertise its directly connected networks.
7. Watch for EIGRP neighbor adjacency messages.
8. Back up each running configuration to the TFTP server at 192.168.12.4.
9. Save each configuration to NVRAM with `write memory`.

## ⚙️ Configuration

### Step 1: Accounts router – passwords and interfaces

![Accounts router CLI showing enable secret, serial and FastEthernet configuration](Accounts_Config.JPG)

*Accounts (Router0): interface addressing, DCE clocking and `enable secret`.*

```
enable
configure terminal
enable secret Accounts@1
interface serial 0/0/0
 ip address 10.0.0.1 255.0.0.0
 clock rate 64000
 no shutdown
interface fastEthernet 0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
```

| Command | Mode | What it does |
|---|---|---|
| `enable` | User EXEC → Privileged EXEC | Enters privileged mode (`Accounts#`). |
| `configure terminal` | Privileged EXEC → Global config | Enters global configuration (`Accounts(config)#`). |
| `enable secret Accounts@1` | Global config | Sets the privileged-mode password, stored as a hash. |
| `interface serial 0/0/0` | Global config → Interface config | Selects the serial interface (`Accounts(config-if)#`). |
| `ip address 10.0.0.1 255.0.0.0` | Interface config | Assigns the IP address and mask. |
| `clock rate 64000` | Interface config | Supplies a 64 kbps clock; DCE end only. |
| `no shutdown` | Interface config | Brings the interface up; router interfaces are shut down by default. |
| `interface fastEthernet 0/0` | Global config → Interface config | Selects the LAN interface. |
| `ip address 192.168.10.1 255.255.255.0` | Interface config | Makes the router the `.1` gateway address of the Accounts LAN. |

**Notes:** The screenshot shows a partially typed `enable se` line before the full `enable secret Accounts@1` command. The Se0/0/0 interface reports `changed state to down` right after `no shutdown`, which is normal while the far-end router is not yet configured. The Fa0/0 interface goes up/up straight away because the switch is connected. The `hostname Accounts` command was entered before this screenshot and is not visible in it.

### Step 2: HR router – hostname, passwords and interfaces

![HR router CLI showing hostname, enable secret and interface configuration](HR_Config.JPG)

*HR (Router1): hostname, `enable secret`, two serial interfaces and the LAN interface.*

```
enable
configure terminal
hostname HR
enable secret HR@1
interface serial 0/0/1
 ip address 10.0.0.2 255.0.0.0
 no shutdown
interface serial 0/0/0
 ip address 11.0.0.1 255.0.0.0
 clock rate 64000
 no shutdown
interface fastEthernet 0/0
 ip address 192.168.11.1 255.255.255.0
 no shutdown
```

| Command | Mode | What it does |
|---|---|---|
| `hostname HR` | Global config | Renames the router; the prompt changes from `Router` to `HR`. |
| `enable secret HR@1` | Global config | Sets the hashed privileged-mode password. |
| `ip address 10.0.0.2 255.0.0.0` (Se0/0/1) | Interface config | DTE side of the Accounts–HR link; the interface came up. |
| `ip address 11.0.0.1 255.0.0.0` (Se0/0/0) | Interface config | Address for the link toward Sales. |
| `clock rate 64000` | Interface config | HR is the DCE end of the HR–Sales link. |
| `ip address 192.168.11.1 255.255.255.0` (Fa0/0) | Interface config | Gateway address of the HR LAN. |

(Other commands as explained in Step 1.)

**Notes:** The screenshot shows one mistyped command, `int se0/0`, which returned `% Invalid interface type and number`, because the interface is `serial 0/0/0` on this router. It was retyped correctly.

### Step 3: Sales router – hostname, passwords and interfaces

![Sales router CLI showing hostname, enable secret and interface configuration](Sales_Config.JPG)

*Sales (Router2): hostname, `enable secret`, serial link to HR and the LAN interface.*

```
enable
configure terminal
hostname Sales
enable secret Sales@1
interface serial 0/0/1
 ip address 11.0.0.2 255.0.0.0
 no shutdown
interface fastEthernet 0/0
 ip address 192.168.12.1 255.255.255.0
 no shutdown
```

**Notes:** `int fa0` was rejected with `% Invalid input detected` because the port number was incomplete; `int fa0/0` was then used. Se0/0/1 on Sales came up as soon as it was enabled because the DCE side (HR Se0/0/0) was already clocking the link. Fa0/0 went up/up.

### Step 4: Accounts router – check routes, then enable EIGRP

![Accounts router: show ip route with connected routes only, then router eigrp 100](Accounts_EIGRP.JPG)

*Before EIGRP: only connected (`C`) and local (`L`) routes. Then EIGRP AS 100 is started.*

```
show ip route
configure terminal
router eigrp 100
 network 10.0.0.0
 network 192.168.10.0
```

| Command | Mode | What it does |
|---|---|---|
| `show ip route` | Privileged EXEC | Displays the routing table. |
| `router eigrp 100` | Global config → Router config | Starts EIGRP in autonomous system 100 (`Accounts(config-router)#`). |
| `network 10.0.0.0` | Router config | Enables EIGRP on interfaces in 10.0.0.0/8 (Se0/0/0) and advertises it. |
| `network 192.168.10.0` | Router config | Enables EIGRP on Fa0/0 and advertises the Accounts LAN. |

**Notes:** The table before EIGRP lists `10.0.0.0/8` and `192.168.10.0/24` as connected, with `/32` local entries for the router's own addresses. These local entries are why IOS prints "variably subnetted, 2 subnets, 2 masks". `Gateway of last resort is not set`, as expected, because no default route exists.

### Step 5: HR router – check routes, then enable EIGRP

![HR router: routing table, router eigrp 100 and a new adjacency message](HR_EIGRP.JPG)

*HR advertises three networks and immediately forms an adjacency with Accounts.*

```
show ip route
configure terminal
router eigrp 100
 network 10.0.0.0
 network 192.168.11.0
 network 11.0.0.0
```

**Notes:** After `network 10.0.0.0`, the console logged:

```
%DUAL-5-NBRCHANGE: IP-EIGRP 100: Neighbor 10.0.0.1 (Serial0/0/1) is up: new adjacency
```

This shows HR and Accounts agree on the AS number and share the 10.0.0.0 link.

### Step 6: Sales router – check routes, then enable EIGRP

![Sales router: routing table, router eigrp 100 and a new adjacency message](Sales_EIGRP.JPG)

*Sales advertises two networks and forms an adjacency with HR.*

```
show ip route
configure terminal
router eigrp 100
 network 11.0.0.0
 network 192.168.12.0
```

**Notes:** After `network 11.0.0.0`, the console logged:

```
%DUAL-5-NBRCHANGE: IP-EIGRP 100: Neighbor 11.0.0.1 (Serial0/0/1) is up: new adjacency
```

### Step 7: Back up each configuration to TFTP and save

![Accounts router copying running-config to TFTP and saving](Accounts_Config_Saving.JPG)

*Accounts: backup of 1009 bytes to 192.168.12.4 as `Accounts_Config`, then saved to NVRAM.*

![HR router copying running-config to TFTP and saving](HR_Config_Saving.JPG)

*HR: backup of 1007 bytes as `HR_Config`, then saved.*

![Sales router copying running-config to TFTP and saving](Sales_Config_Saving.JPG)

*Sales: backup of 988 bytes as `Sales_Config`, then saved.*

```
copy running-config tftp
  Address or name of remote host []? 192.168.12.4
  Destination filename [<hostname>-confg]? <Hostname>_Config
write memory
```

| Command | Mode | What it does |
|---|---|---|
| `copy running-config tftp` | Privileged EXEC | Copies the active configuration to a TFTP server; the router prompts for the server address and filename. |
| `write memory` (`wr`) | Privileged EXEC | Saves the running configuration to startup configuration (NVRAM); IOS shows `Building configuration... [OK]`. |

**Notes:** `!!` in the output means data blocks were acknowledged by the server. The router that performs the copy must have a route to 192.168.12.4 and the server must have TFTP enabled. Accounts and HR are on other subnets from the server, so their successful copies show that a path to the Sales LAN existed at that point.

## ✅ Verification & Results

| Check | Command / Action | What success looks like | Result |
|---|---|---|---|
| Interfaces up and addressed | `show ip route` | `C` and `L` entries for each configured interface | Shown in screenshot (all three routers) |
| EIGRP neighbors form (HR–Accounts) | Console log / `show ip eigrp neighbors` | `%DUAL-5-NBRCHANGE ... Neighbor 10.0.0.1 ... is up` | Shown in screenshot (HR) |
| EIGRP neighbors form (Sales–HR) | Console log / `show ip eigrp neighbors` | `%DUAL-5-NBRCHANGE ... Neighbor 11.0.0.1 ... is up` | Shown in screenshot (Sales) |
| Routes learned via EIGRP | `show ip route eigrp` | `D` entries for the remote LANs | Expected (not captured) |
| End-to-end connectivity | `ping` from a PC in one LAN to a PC in another | Replies from the far PC | Expected (not captured) |
| Backup to TFTP | `copy running-config tftp` | `Writing running-config....!!` and `[OK - n bytes]` | Shown in screenshot (all three routers) |
| Config saved | `write memory` | `Building configuration... [OK]` | Shown in screenshot (all three routers) |

**Expected EIGRP routes (not captured; metrics omitted).** With all networks advertised, each router should learn the LANs and the serial link it is not directly attached to:

| Router | Expected `D` routes | Next hop |
|---|---|---|
| Accounts | 11.0.0.0/8, 192.168.11.0/24, 192.168.12.0/24 | 10.0.0.2 (HR) |
| HR | 192.168.10.0/24 and 192.168.12.0/24 | 10.0.0.1 (Accounts) and 11.0.0.2 (Sales) |
| Sales | 10.0.0.0/8, 192.168.10.0/24, 192.168.11.0/24 | 11.0.0.1 (HR) |

To reproduce the checks, run on any router:

```
show ip eigrp neighbors
show ip route eigrp
show ip protocols
show ip interface brief
```

## 🛠️ Troubleshooting Notes

**Issues seen in the screenshots**

| Symptom | Cause | Fix |
|---|---|---|
| `% Invalid interface type and number` on HR | Typed `int se0/0`; this router's serial ports are numbered `0/0/0` and `0/0/1` | Use `interface serial 0/0/0` |
| `% Invalid input detected at '^' marker` on Sales | Typed `int fa0` without the slot/port | Use `interface fastEthernet 0/0` |
| Serial interface `changed state to down` after `no shutdown` | Far-end router not yet configured or enabled | Configure the other end; expect it to come up |

**General EIGRP checklist (not specific to this lab's screenshots)**

| Symptom | Likely cause | How to check | Fix |
|---|---|---|---|
| No EIGRP neighbors | AS number mismatch, interface not covered by a `network` statement, or link down | `show ip protocols`, `show ip interface brief` | Match the AS and add the missing `network` line |
| Serial line protocol down | Missing `clock rate` on the DCE end, or one end shut down | `show controllers serial 0/0/0`, `show ip interface brief` | Add `clock rate` on the DCE side and `no shutdown` |
| Neighbor up but remote LAN missing | `network` statement not entered for that LAN | `show ip route eigrp` on the far router | Add the LAN's `network` command |
| Ping fails between LANs | Wrong host IP, mask or gateway | `ipconfig` on the PC | Set the gateway to the router's Fa0/0 address |
| TFTP copy fails (`.` or timeout) | No route to server, wrong server IP or TFTP service off | `ping 192.168.12.4`, check Server → Services → TFTP | Fix routing or enable the service |

## 💼 Skills Demonstrated

- Configured EIGRP (AS 100) on three routers and confirmed neighbor adjacencies from console messages.
- Addressed router interfaces and set up serial point-to-point links with DCE clocking.
- Used `enable secret` to protect privileged access on every router.
- Read routing tables and interpreted connected (`C`), local (`L`) and EIGRP (`D`) entries.
- Backed up device configurations to a TFTP server and saved them to NVRAM.
- Planned a multi-site addressing scheme using /24 LANs and separate serial subnets.
- Diagnosed and corrected CLI errors (invalid interface names).

**CCNA 200-301 mapping:** 1.0 Network Fundamentals (topology, IPv4 addressing, cabling); 3.0 IP Connectivity (routing table components, administrative distance, EIGRP concepts); 4.0 IP Services (TFTP file transfer); 5.0 Security Fundamentals (device access control with `enable secret`).

## 📂 Files in This Lab

```
EIGRP Lab/
├── README.md
├── EIGRP_Lab.pkt              # Packet Tracer project file
├── Topology.JPG               # Network topology
├── Accounts_Config.JPG        # Accounts interface and password config
├── Accounts_EIGRP.JPG         # Accounts routing table and EIGRP config
├── Accounts_Config_Saving.JPG # Accounts TFTP backup and save
├── HR_Config.JPG              # HR interface and password config
├── HR_EIGRP.JPG               # HR routing table and EIGRP config
├── HR_Config_Saving.JPG       # HR TFTP backup and save
├── Sales_Config.JPG           # Sales interface and password config
├── Sales_EIGRP.JPG            # Sales routing table and EIGRP config
└── Sales_Config_Saving.JPG    # Sales TFTP backup and save
```

## 🚀 How to Open This Lab

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco NetAcad account).
2. Open `EIGRP_Lab.pkt`.
3. Click a router, open the **CLI** tab, and press Enter. Use the `enable secret` for that router (see the topology labels).
4. Run `show ip eigrp neighbors` and `show ip route` to see the EIGRP results.
5. To repeat the backups, make sure the TFTP service is turned on in the server's **Services** tab.

## 🔭 Possible Improvements

- Correct the host and server default gateways to each LAN's router address (…1) and test ping across all three LANs.
- Use `passive-interface fastEthernet 0/0` on each router so EIGRP Hellos are not sent into user LANs.
- Replace the /8 serial links with /30 subnets, and move the 11.0.0.0/8 link to private address space.
- Add console and VTY passwords, `service password-encryption`, and SSH instead of relying on `enable secret` alone.
- Add EIGRP authentication and test a link failure to watch the routes reconverge.

## 👤 Author

Aspiring Network Engineer, building CCNA / networking fundamentals skills.

---
*This lab is part of my ongoing CCNA / Network Administrator study roadmap.*
