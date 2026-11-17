[← Back to all labs](../README.md)

# VLAN Configuration and Trunk Port Between Two Switches (Cisco Packet Tracer)

This lab creates two VLANs (Sales and HR) on two switches, assigns PCs to them through access ports, and connects the switches with a trunk port so both VLANs travel across the link on a single cable.

## 📋 At a Glance

| | |
|---|---|
| **Lab number** | _(add lab number)_ |
| **Level** | Beginner, CCNA |
| **Tools** | Cisco Packet Tracer, Cisco IOS CLI |
| **Devices** | 2 switches (SW0, SW1), 4 PCs (PC0 to PC3). Switch model not visible in screenshots. |
| **Protocols / features** | VLANs (802.1Q), access ports, trunk port |
| **Network(s)** | 10.0.0.0/24 (subnet mask assumed, see Subnet Analysis) |
| **CCNA domains** | 2.0 Network Access, 1.0 Network Fundamentals |

## 📌 Project Objective

Configure two VLANs on both switches and carry them across a trunk link so that:

- PCs in the same VLAN can communicate across the trunk (PC0 and PC2 in VLAN 2, PC1 and PC3 in VLAN 3).
- PCs in different VLANs are kept apart at Layer 2 (PC0 and PC1 cannot communicate).

## 🌍 Real-World Scenario

Offices often place several departments on the same physical network. VLANs keep Sales and HR traffic separate on shared switches, which limits broadcast traffic and makes access control easier. A trunk link lets one VLAN design span several switches, so a department's computers can sit on any switch in the building and still belong to the same VLAN.

## 🖧 Topology

![Network topology showing two switches joined by a trunk port, with PC0 and PC2 in VLAN 2 (Sales) and PC1 and PC3 in VLAN 3 (HR)](Images/Trunk.JPG)

The two switches are joined by a trunk link on Fa0/3. Each switch has two PCs connected to Fa0/1 and Fa0/2. The diagram labels the trunk port and states that VLAN 2 is for the Sales department and VLAN 3 is for the HR department.

Port mapping comes from the configuration screenshots, which assign:

- **SW0:** Fa0/1 to VLAN 2 (PC0), Fa0/2 to VLAN 3 (PC1)
- **SW1:** Fa0/1 to VLAN 2 (PC2), Fa0/2 to VLAN 3 (PC3)

### Cabling

| From | Port | To | Port | Cable type |
|---|---|---|---|---|
| PC0 | Fa0 | SW0 | Fa0/1 | Straight-through (PC to switch, unlike devices) |
| PC1 | Fa0 | SW0 | Fa0/2 | Straight-through (PC to switch, unlike devices) |
| SW0 | Fa0/3 | SW1 | Fa0/3 | Crossover by standard rule (switch to switch, like devices). Cable type not visible in screenshots. |
| PC2 | Fa0 | SW1 | Fa0/1 | Straight-through (PC to switch, unlike devices) |
| PC3 | Fa0 | SW1 | Fa0/2 | Straight-through (PC to switch, unlike devices) |

Packet Tracer's auto-MDIX can make the difference between straight-through and crossover invisible, so the cable types above follow the standard rule.

### Addressing Table

| Device | Interface | IP Address | Subnet Mask | Prefix | Default Gateway |
|---|---|---|---|---|---|
| PC0 | Fa0 | 10.0.0.10 | 255.255.255.0 (assumed) | /24 | None set (no router in this topology) |
| PC1 | Fa0 | 10.0.0.20 | 255.255.255.0 (assumed) | /24 | None set (no router in this topology) |
| PC2 | Fa0 | 10.0.0.30 | 255.255.255.0 (assumed) | /24 | None set (no router in this topology) |
| PC3 | Fa0 | 10.0.0.40 | 255.255.255.0 (assumed) | /24 | None set (no router in this topology) |

No IP address configuration for SW0 or SW1 appears in the screenshots, so none is listed.

### Subnet Analysis

| Network | Mask | Network Address | First Usable | Last Usable | Broadcast | Usable Hosts | Private (RFC 1918)? |
|---|---|---|---|---|---|---|---|
| 10.0.0.0/24 | 255.255.255.0 | 10.0.0.0 | 10.0.0.1 | 10.0.0.254 | 10.0.0.255 | 254 | Yes (within 10.0.0.0/8) |

All four PCs use addresses from this single /24. The VLANs separate them at Layer 2, so PCs in different VLANs share a subnet but still cannot reach each other. Giving each VLAN its own subnet is the usual design, and it is listed under Possible Improvements.

## 📚 Concepts Behind This Lab

### VLANs create separate broadcast domains

A VLAN is a logical group of switch ports that behaves like its own LAN. Broadcasts sent in VLAN 2 do not reach VLAN 3, even when both VLANs use the same physical switches. In this lab, VLAN 2 is named Sales and VLAN 3 is named HR. Each switch needs the VLANs defined, because VLAN names are stored locally on each switch unless VTP is used.

### Access ports and trunk ports serve different purposes

| | Access port | Trunk port |
|---|---|---|
| **Purpose** | Connects one end device (a PC) | Carries several VLANs between switches or to a router |
| **VLANs carried** | One VLAN | All allowed VLANs |
| **Frame format** | Untagged | Tagged with IEEE 802.1Q, except the native VLAN |
| **Config in this lab** | `switchport mode access`, `switchport access vlan X` | `switchport mode trunk` |

### 802.1Q tagging and the native VLAN

A trunk adds a 4-byte 802.1Q tag to each frame so the receiving switch knows which VLAN the frame belongs to. Frames from the native VLAN (VLAN 1 by default) travel untagged. Both ends of a trunk must agree on the native VLAN, or traffic can leak between VLANs. This lab does not change the native VLAN, so it stays at the default.

### Trunk negotiation

Cisco switches use DTP (Dynamic Trunking Protocol) to negotiate trunks. Many switches default to `dynamic auto`, which forms a trunk only if the other end asks for one. Setting `switchport mode trunk` forces the port into trunk mode and is the clearer choice for a lab. The screenshots show the trunk command on SW0 only. The SW1 trunk configuration is not shown.

### Same subnet across VLANs does not allow communication

Two PCs in different VLANs cannot exchange frames at Layer 2, regardless of IP address. Communication between VLANs requires a Layer 3 device, such as a router using router-on-a-stick or a Layer 3 switch with SVIs. This lab has no router, so the expected result is that PCs in VLAN 2 and VLAN 3 cannot reach each other.

## 🧭 Lab Steps

1. Create VLAN 2 (named Sales) and VLAN 3 (named HR) on SW0.
2. Assign Fa0/1 to VLAN 2 and Fa0/2 to VLAN 3 on SW0 as access ports.
3. Configure Fa0/3 on SW0 as a trunk port.
4. Create VLAN 2 (Sales) and VLAN 3 (HR) on SW1.
5. Assign Fa0/1 to VLAN 2 and Fa0/2 to VLAN 3 on SW1 as access ports.
6. Verify the VLAN and trunk configuration and test connectivity between PCs.

## ⚙️ Configuration

### Step 1: Create VLANs and assign access ports on SW0

![SW0 CLI showing VLAN creation and access port assignment, with one invalid command corrected](Images/SW0_Vlan_Config.JPG)

```
enable
configure terminal
vlan 2
 name Sales
 exit
vlan 3
 name HR
 exit
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 2
 exit
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 3
 exit
exit
```

| Command | Mode | What it does |
|---|---|---|
| `enable` | User EXEC (`Switch>`) | Enters privileged EXEC mode (`Switch#`) |
| `configure terminal` (`conf t`) | Privileged EXEC | Enters global configuration mode |
| `vlan 2` | Global config | Creates VLAN 2 and opens VLAN configuration mode |
| `name Sales` | VLAN config (`config-vlan`) | Gives VLAN 2 a descriptive name |
| `exit` | VLAN config | Returns to global configuration mode |
| `vlan 3` / `name HR` | Global config / VLAN config | Creates VLAN 3 and names it HR |
| `interface fastEthernet 0/1` (`int f0/1`) | Global config | Opens the configuration for port Fa0/1 (`config-if`) |
| `switchport mode access` | Interface config | Sets the port to access mode, which carries one VLAN untagged |
| `switchport access vlan 2` | Interface config | Places the port in VLAN 2 |
| `exit` (from interface mode) | Interface config | Returns to global configuration mode |
| `exit` (from global mode) | Global config | Returns to privileged EXEC mode |

**Notes:**

- `switchport mode access` and `switchport access vlan` are two separate commands. The screenshot shows an attempt to combine them into `switchport mode access vlan 2`, which IOS rejects with `% Invalid input detected at '^' marker.` The corrected command is `switchport access vlan 2`, entered after `switchport mode access`.
- The VLANs must exist before the ports are assigned to them, so the VLAN commands come first.
- The prompt shows `Switch` because the hostname was not changed. Setting `hostname SW0` is a helpful habit when documenting labs with several switches.

### Step 2: Configure the trunk port on SW0

![SW0 CLI showing Fa0/3 configured as a trunk and the interface line protocol changing state](Images/SW0_Trunk_Config.JPG)

```
interface fastEthernet 0/3
 switchport mode trunk
 exit
```

| Command | Mode | What it does |
|---|---|---|
| `interface fastEthernet 0/3` (`int f0/3`) | Global config | Opens the configuration for port Fa0/3 |
| `switchport mode trunk` | Interface config | Forces the port into trunk mode so it carries multiple VLANs |

**Notes:**

- The console shows `%LINEPROTO-5-UPDOWN` messages for Fa0/3, first `changed state to down` and then `changed state to up`. This is normal. The port resets when its mode changes, and the link comes back up within a few seconds.
- Trunking carries all VLANs by default, so the VLAN list does not need to be restricted in this lab.

### Step 3: Configure VLANs and access ports on SW1

![SW1 CLI showing VLAN creation and access port assignment, with one invalid command corrected](Images/SW1_Vlan_Config.JPG)

```
enable
configure terminal
vlan 2
 name Sales
 exit
vlan 3
 name HR
 exit
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 2
 exit
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 3
 exit
exit
```

**Notes:**

- The same invalid `switchport mode access vlan 2` error appears on SW1 and was corrected the same way.
- VLAN names and IDs match SW0. Switches do not share VLAN names automatically, so each switch needs its own VLAN commands.
- The screenshots do not show a `switchport mode trunk` command on SW1's Fa0/3. The trunk must be set on both ends for reliable operation, so confirm this on SW1 before testing.

## ✅ Verification & Results

| Check | Command / Action | What success looks like | Result |
|---|---|---|---|
| VLANs created with names | Commands entered on SW0 and SW1 | Commands accepted without errors, except the one corrected in each case | Shown in screenshot (SW0 and SW1 commands). `show vlan brief` not captured. |
| Trunk port comes up | `switchport mode trunk` on SW0 Fa0/3 | Line protocol goes from down to up | Shown in screenshot (`changed state to down`, then `changed state to up`) |
| VLAN membership | `show vlan brief` on SW0 and SW1 | VLAN 2 Sales has Fa0/1, VLAN 3 HR has Fa0/2 | Expected (not captured) |
| Trunk operation | `show interfaces trunk` on SW0 | Fa0/3 listed as trunking, native VLAN 1 | Expected (not captured) |
| Same-VLAN ping across trunk | PC0 to PC2 (10.0.0.10 to 10.0.0.30) | Replies received, success rate 100% | Expected (not captured) |
| Same-VLAN ping across trunk | PC1 to PC3 (10.0.0.20 to 10.0.0.40) | Replies received, success rate 100% | Expected (not captured) |
| Cross-VLAN isolation | PC0 to PC1 (10.0.0.10 to 10.0.0.20) | No reply, because the VLANs are separated and there is no router | Expected (not captured) |

No ping results were captured in screenshots. The rows marked "Expected" describe what the reader should see when repeating the tests.

**Expected output: `show vlan brief` on SW0** (simplified, not captured from this lab)

```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------
1    default                          active    (remaining ports)
2    Sales                            active    Fa0/1
3    HR                               active    Fa0/2
```

**Expected output: ping PC0 to PC1 (different VLANs)** (not captured from this lab)

```
Pinging 10.0.0.20 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 10.0.0.20:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

PC0 and PC1 share the same subnet, so PC0 sends an ARP request for PC1. The request never crosses the VLAN boundary, so PC0 never receives a reply.

## 🛠️ Troubleshooting Notes

### Problems encountered in this lab

- **Invalid command on access ports (SW0 and SW1).** Entering `switchport mode access vlan 2` produced `% Invalid input detected at '^' marker.` The fix was to enter `switchport access vlan 2` as a separate command after `switchport mode access`. IOS places the caret under the first word it does not recognize, so reading the caret position is the quickest way to find the error.
- **Trunk port flapped.** Fa0/3 went down and back up when its mode changed. This is expected and does not indicate a fault.

### General troubleshooting for this lab type

The table below lists the most likely failures in a VLAN and trunk lab. These are general tips, not problems recorded from this lab.

| Symptom | Likely cause | How to check | Fix |
|---|---|---|---|
| PCs in the same VLAN cannot ping across the trunk | Trunk not up, or the VLAN is missing on one switch | `show interfaces trunk`, `show vlan brief` on both switches | Confirm `switchport mode trunk` on both ends, and create the VLAN on the switch that lacks it |
| A PC is in the wrong VLAN | Port assigned to the wrong VLAN | `show vlan brief`, `show running-config` | `switchport access vlan <correct ID>` on that port |
| Trunk port stays down | Cable problem, or the far end is down | `show ip interface brief`, check link lights | Check the cable and the far-end port |
| `% Invalid input detected` | Typo or incorrect command syntax | Look at the caret (`^`) position | Enter the commands one at a time and use `?` for help |
| Ping fails between VLANs | Expected behavior without routing | Ping within a VLAN to confirm the switches work | Add inter-VLAN routing (see Possible Improvements) |

## 💼 Skills Demonstrated

- Created and named two VLANs (Sales and HR) on two switches.
- Assigned access ports to VLANs so each PC joins the correct department network.
- Configured an 802.1Q trunk to carry multiple VLANs between switches.
- Read IOS error messages and corrected an invalid command.
- Planned and documented the addressing and port layout for a two-switch network.

**CCNA 200-301 mapping:** 2.0 Network Access (VLANs, trunk ports and 802.1Q, access port configuration). 1.0 Network Fundamentals (IPv4 addressing and subnetting).

## 📂 Files in This Lab

```
VLAN-Trunking-Lab/
├── README.md
└── Images/
    ├── Trunk.JPG                 # Topology diagram
    ├── SW0_Trunk_Config.JPG      # SW0 trunk port configuration
    ├── SW0_Vlan_Config.JPG       # SW0 VLAN and access port configuration
    └── SW1_Vlan_Config.JPG       # SW1 VLAN and access port configuration
```

## 🚀 How to Open This Lab

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco NetAcad account).
2. Open the Packet Tracer (`.pkt`) file for this lab, if included in the folder.
3. Click SW0, open the **CLI** tab, and run `show vlan brief` and `show interfaces trunk`. Repeat on SW1.
4. Click PC0, open the **Desktop** tab, then **Command Prompt**, and run `ping 10.0.0.30`. Then run `ping 10.0.0.20` to see the cross-VLAN result.

## 🔭 Possible Improvements

- **Add inter-VLAN routing.** A router-on-a-stick configuration on a router connected to the trunk would let VLAN 2 and VLAN 3 communicate.
- **Give each VLAN its own subnet.** Using a separate /24 for each VLAN (for example, 10.0.2.0/24 for Sales and 10.0.3.0/24 for HR) matches standard design practice. The current setup places all four PCs in one subnet.
- **Harden the trunk.** Restrict allowed VLANs with `switchport trunk allowed vlan 2,3`, move the native VLAN away from VLAN 1, and turn off DTP with `switchport nonegotiate`.
- **Rename the switches.** Use `hostname SW0` and `hostname SW1` so the screenshots and configs are easier to read.

## 👤 Author

[Your Name] — Aspiring Network Engineer, building CCNA / networking fundamentals skills.

---
*This lab is part of my ongoing CCNA / Network Administrator study roadmap.*
