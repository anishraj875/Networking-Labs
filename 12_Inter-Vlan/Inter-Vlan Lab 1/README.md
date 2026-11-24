[← Back to all labs](../README.md)

# Inter-VLAN Routing with One Router Interface per VLAN (Cisco Packet Tracer)

This lab places two PCs in separate VLANs on one switch, then uses a router with one interface per VLAN to route traffic between them. It shows why hosts in different VLANs cannot talk without a Layer 3 device, and how a router interface becomes the default gateway for each VLAN.

## 📋 At a Glance

| | |
|---|---|
| **Lab number** | [N] |
| **Level** | Beginner, CCNA |
| **Tools** | Cisco Packet Tracer, Cisco IOS CLI |
| **Devices** | 1 router (model not shown), 1 switch (model not shown), 2 PCs |
| **Protocols / features** | VLANs, access ports, inter-VLAN routing, IPv4 addressing, default gateway |
| **Network(s)** | 192.168.1.0/24 (VLAN 10, HR), 172.168.0.0/16 (VLAN 20, MR) |
| **CCNA domains** | Network Access (2.0), IP Connectivity (3.0), Network Fundamentals (1.0) |

## 📌 Project Objective

Create two VLANs on SW1, assign each PC to its VLAN, and connect the switch to the router with one link per VLAN. Configure the router with one IP address per VLAN so that each PC can use its VLAN's router interface as its default gateway. Success means PCs in different VLANs can reach each other through R1.

## 🌍 Real-World Scenario

Organizations split users into VLANs by department, for example HR and marketing, to keep broadcast traffic contained and to control which groups can reach each other. Each VLAN is its own subnet, and a Layer 3 device (a router or a Layer 3 switch) moves traffic between them. Real networks often use a single trunk link with subinterfaces (router-on-a-stick) instead of one physical link per VLAN, and they add access control lists to decide which VLANs may communicate.

## 🖧 Topology

![Inter-VLAN topology diagram showing SW1 connected to PC0 (VLAN 10, HR), PC1 (VLAN 20, MR) and router R1 through Gig0/1 and Gig0/0](Inter-Vlan_Topology.JPG)

The switch SW1 connects to two PCs on access ports, Fa0/1 for PC0 (VLAN 10, HR) and Fa0/2 for PC1 (VLAN 20, MR). Two more switch ports link to the router: Fa0/4 goes to R1 Gig0/1, which serves VLAN 10, and Fa0/3 goes to R1 Gig0/0, which serves VLAN 20. The labels on the diagram match the configuration file.

### Cabling

| From | Port | To | Port | Cable type |
|---|---|---|---|---|
| SW1 | Fa0/1 | PC0 | Fa0 | Straight-through (copper) |
| SW1 | Fa0/2 | PC1 | Fa0 | Straight-through (copper) |
| SW1 | Fa0/4 | R1 | Gig0/1 | Straight-through (copper) |
| SW1 | Fa0/3 | R1 | Gig0/0 | Straight-through (copper) |

The cable type is not labeled on the diagram, so these follow the standard rule: a switch and a PC, or a switch and a router, are unlike devices and use straight-through cable.

### Addressing Table

| Device | Interface | IP Address | Subnet Mask | Prefix | Default Gateway |
|---|---|---|---|---|---|
| R1 | GigabitEthernet0/1 (Gig0/1) | 192.168.1.1 | 255.255.255.0 | /24 | Not applicable (router interface) |
| R1 | GigabitEthernet0/0 (Gig0/0) | 172.168.1.1 | 255.255.0.0 | /16 | Not applicable (router interface) |
| PC0 (HR) | Fa0 | 192.168.1.10 | 255.255.255.0 | /24 | 192.168.1.1 |
| PC1 (MR) | Fa0 | 172.168.1.20 | 255.255.0.0 | /16 | 172.168.1.1 |

**Note on PC1:** the configuration file lists PC1 as 172.168.1.20, but the topology label reads 172.168.1.10. This README uses 172.168.1.20 from the configuration file. Confirm which address was used in the Packet Tracer file.

### Subnet Analysis

| Network | Mask | Network Address | First Usable | Last Usable | Broadcast | Usable Hosts | Private (RFC 1918)? |
|---|---|---|---|---|---|---|---|
| VLAN 10 (HR) | 255.255.255.0 (/24) | 192.168.1.0 | 192.168.1.1 | 192.168.1.254 | 192.168.1.255 | 254 | Yes (192.168.0.0/16) |
| VLAN 20 (MR) | 255.255.0.0 (/16) | 172.168.0.0 | 172.168.0.1 | 172.168.255.254 | 172.168.255.255 | 65,534 | No (see note below) |

Each VLAN has its own subnet, and the router interface on that VLAN holds the first usable address (.1) as the default gateway. The gateway addresses sit inside their own subnets, so each PC can reach its gateway directly.

**Address range note:** 172.168.0.0/16 is not a private range. RFC 1918 reserves 172.16.0.0 to 172.31.255.255 (172.16.0.0/12), and 172.168 falls outside that block. The lab works in Packet Tracer, but in a real network this range would be publicly routable. A private address such as 172.16.1.0/24 would follow the standard. This is a good point to discuss in an interview.

## 📚 Concepts Behind This Lab

**Why VLANs need a router.** A VLAN is a separate broadcast domain at Layer 2, and each VLAN uses its own subnet. A switch forwards frames only within a VLAN, so PC0 (VLAN 10) cannot reach PC1 (VLAN 20) through the switch alone. Traffic between subnets must go to a default gateway, and the router is the device that looks up the destination subnet and forwards the packet into the other VLAN.

**Access ports.** An access port belongs to one VLAN and carries untagged frames for that VLAN only. Each PC is connected to an access port, so `switchport mode access` with `switchport access vlan X` is correct for the PC ports. The router-facing ports on this lab are also assigned to one VLAN each, because each router interface serves one VLAN.

**Connected routes.** The router learns the 192.168.1.0/24 and 172.168.0.0/16 networks automatically from its own interfaces, which are up and have IP addresses. No routing protocol or static route is needed. The router shows these as `C` (connected) entries in its routing table.

**Default gateway.** A PC sends traffic for any address outside its own subnet to its default gateway. PC0 sends traffic for 172.168.0.0/16 to 192.168.1.1, and PC1 sends traffic for 192.168.1.0/24 to 172.168.1.1. If the gateway is wrong, same-subnet traffic still works, but cross-VLAN traffic fails.

**Two ways to route between VLANs.** The table compares the two common designs. This lab uses the first.

| | One router interface per VLAN (this lab) | Router-on-a-stick |
|---|---|---|
| Physical links | One link per VLAN | One trunk link for all VLANs |
| Switch port mode | Access port for each router link | Trunk port toward the router |
| Router configuration | Plain IP address on each physical interface | Subinterface per VLAN with `encapsulation dot1Q` |
| Scales to many VLANs? | Poorly; needs a router port for each VLAN | Well; one link carries all VLANs |
| Good for | Learning the core idea, small networks | Real networks with many VLANs |

**Subnet mask consistency.** A host uses its mask to decide whether a destination is local. If PC1 used 255.255.255.0 while the router used 255.255.0.0, PC1 and the router would disagree about which addresses are on the same network. The masks in this lab match each VLAN's design.

## 🧭 Lab Steps

1. Build the topology: one switch, one router, two PCs, and four straight-through cables as listed in the Cabling table.
2. On SW1, create VLAN 10 (named HR) and VLAN 20 (named MR).
3. On SW1, assign Fa0/1 to VLAN 10 and Fa0/2 to VLAN 20 for the PCs.
4. On SW1, assign Fa0/4 to VLAN 10 for the link to R1 Gig0/1, and Fa0/3 to VLAN 20 for the link to R1 Gig0/0.
5. On R1, configure Gig0/1 with 192.168.1.1/24 and Gig0/0 with 172.168.1.1/16, and bring both interfaces up.
6. Set each PC's IP address, mask and default gateway from the Addressing Table.
7. Verify VLAN membership, interface status, the routing table, and end-to-end pings between the two VLANs.

## ⚙️ Configuration

### Step 1: Create the VLANs and assign switch ports

The full command list is in [inter-Vlan_Configurations.txt](inter-Vlan_Configurations.txt). The tidied version below shows the commands used for this stage.

```
enable
configure terminal
hostname SW1
vlan 10
 name HR
vlan 20
 name MR
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20
interface fastEthernet 0/4
 description SW-to-R1_For-Vlan10
 switchport access vlan 10
interface fastEthernet 0/3
 description SW-to-R1_For-Vlan20
 switchport access vlan 20
end
```

| Command | Mode | What it does |
|---|---|---|
| `enable` | User EXEC | Enters privileged EXEC mode (`SW1#`) |
| `configure terminal` | Privileged EXEC | Enters global configuration mode (`SW1(config)#`) |
| `hostname SW1` | Global config | Sets the switch name shown in the prompt |
| `vlan 10` | Global config | Creates VLAN 10 and enters VLAN configuration mode |
| `name HR` | VLAN config | Gives VLAN 10 a descriptive name |
| `vlan 20` / `name MR` | Global config / VLAN config | Creates and names VLAN 20 |
| `interface fastEthernet 0/1` | Global config | Enters configuration for port Fa0/1 (`SW1(config-if)#`) |
| `switchport mode access` | Interface config | Makes the port a single-VLAN access port, not a trunk |
| `switchport access vlan 10` | Interface config | Places the port in VLAN 10 |
| `description ...` | Interface config | Labels the port for documentation; does not change behavior |
| `end` | Any config mode | Returns to privileged EXEC mode |

**Notes:**
- The configuration file uses `host SW1`. The IOS command is `hostname SW1`, so the tidied version above uses `hostname`.
- Ports Fa0/3 and Fa0/4 are assigned to a VLAN without `switchport mode access`. On many switches the default mode is dynamic auto, so adding `switchport mode access` to these two ports is recommended to fix the port as an access port.
- VLAN names are optional for the function of the lab, but they document the design and help when reading `show vlan brief`.

### Step 2: Configure the router interfaces

```
enable
configure terminal
hostname R1
interface GigabitEthernet0/1
 ip address 192.168.1.1 255.255.255.0
 no shutdown
interface GigabitEthernet0/0
 ip address 172.168.1.1 255.255.0.0
 no shutdown
end
```

| Command | Mode | What it does |
|---|---|---|
| `hostname R1` | Global config | Names the router |
| `interface GigabitEthernet0/1` | Global config | Enters configuration for the link to SW1 Fa0/4 (VLAN 10) |
| `ip address 192.168.1.1 255.255.255.0` | Interface config | Assigns the VLAN 10 gateway address and mask |
| `no shutdown` | Interface config | Brings the interface up; router interfaces start administratively down |
| `interface GigabitEthernet0/0` | Global config | Enters configuration for the link to SW1 Fa0/3 (VLAN 20) |
| `ip address 172.168.1.1 255.255.0.0` | Interface config | Assigns the VLAN 20 gateway address and mask |

**Notes:**
- Router interfaces are shut down by default, so `no shutdown` is required on each one. Without it, the interface stays `administratively down`.
- The configuration file lists the interfaces as `g0/1` and `g0/0`. The full names above (`GigabitEthernet0/1`, `GigabitEthernet0/0`) are the same interfaces.
- Each router interface is a separate broadcast domain connected to one VLAN, so the router needs no routing protocol for these two directly connected networks.

### Step 3: Configure the PCs

The file gives the values for each PC. Set them in each PC's Desktop, under IP Configuration.

| PC | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC0 (HR) | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC1 (MR) | 172.168.1.20 | 255.255.0.0 | 172.168.1.1 |

**Notes:** The default gateway must be the router interface on the PC's own VLAN. PC0 uses Gig0/1 and PC1 uses Gig0/0. Using the other VLAN's gateway would send traffic to the wrong interface.

## ✅ Verification & Results

| Check | Command / Action | What success looks like | Result |
|---|---|---|---|
| VLAN membership | `show vlan brief` on SW1 | VLAN 10 (HR) lists Fa0/1 and Fa0/4; VLAN 20 (MR) lists Fa0/2 and Fa0/3 | Expected (not captured) |
| Switch port mode | `show interfaces status` on SW1 | Fa0/1 to Fa0/4 show connected and the correct VLAN | Expected (not captured) |
| Router interface state | `show ip interface brief` on R1 | Gig0/1 and Gig0/0 show up/up with the addresses above | Expected (not captured) |
| Routing table | `show ip route` on R1 | Two `C` (connected) entries, one per VLAN network | Expected (not captured) |
| PC settings | `ipconfig` on PC0 and PC1 | Address, mask and gateway match the Addressing Table | Expected (not captured) |
| Gateway reachability | `ping 192.168.1.1` from PC0; `ping 172.168.1.1` from PC1 | Replies from each PC's own gateway | Expected (not captured) |
| Inter-VLAN ping | `ping 172.168.1.20` from PC0; `ping 192.168.1.10` from PC1 | Replies in both directions, through R1 | Expected (not captured) |
| Path check | `tracert 172.168.1.20` from PC0 | Hop 1 is 192.168.1.1 (R1), then the destination | Expected (not captured) |

No screenshots of command output were provided, so every result above is expected and not captured.

**Expected output: `show ip route` on R1**

```
C    172.168.0.0/16 is directly connected, GigabitEthernet0/0
C    192.168.1.0/24 is directly connected, GigabitEthernet0/1
```

**Expected output: `ping` from PC0 to PC1**

```
Pinging 172.168.1.20 with 32 bytes of data:
Reply from 172.168.1.20: bytes=32 time<1ms TTL=127
```

Packet Tracer often shows a timeout on the first ping while ARP resolves. Later pings succeed. The TTL drops from 128 to 127 because the packet crosses one router hop, which confirms routing took place.

## 🛠️ Troubleshooting Notes

No problems were recorded in the files provided. The table below lists the most likely failures for this lab and how to check each one, working from Layer 1 upward.

| Symptom | Likely cause | How to check | Fix |
|---|---|---|---|
| Router interface shows administratively down | `no shutdown` missing | `show ip interface brief` on R1 | Enter the interface and run `no shutdown` |
| PC cannot ping its own gateway | Wrong switch port or wrong VLAN on that port | `show vlan brief` on SW1 | Move the cable or change `switchport access vlan` to match the PC's VLAN |
| PC can reach its gateway but not the other VLAN | Wrong default gateway on the PC | `ipconfig` on the PC | Set the gateway to the router interface on the PC's VLAN |
| Cross-VLAN ping fails in one direction only | Gateway or mask error on one PC | Compare `ipconfig` to the Addressing Table | Correct the mask or gateway on that PC |
| Ping to the gateway works, ping to the other PC does not | Mask on one PC does not match the other PC's network | `ipconfig /all` on both PCs | Set each PC to the mask in the Addressing Table |
| A PC shows an address from the wrong VLAN | Cable plugged into a port assigned to the other VLAN | Trace the cable and check `show vlan brief` | Plug the PC into the correct access port |
| Router has no route for one network | An interface is down or has no IP address | `show ip route` on R1 | Check `show ip interface brief` and re-enter the address |

**General tip:** Ping in this order to find the failure point: the PC's own address, then its gateway, then the PC on the other VLAN. The first failed step shows the layer to examine.

## 💼 Skills Demonstrated

- Created and named VLANs on a Layer 2 switch and assigned access ports to them.
- Configured one routed interface per VLAN on a router, giving each VLAN its own default gateway.
- Planned and verified IPv4 addressing for two subnets with different masks (/24 and /16).
- Explained why hosts in different VLANs cannot communicate without a Layer 3 device.
- Documented the topology, cabling and addressing in a repeatable format.

**CCNA 200-301 mapping:** 2.0 Network Access (VLAN configuration, access ports); 1.0 Network Fundamentals (IPv4 addressing and subnetting); 3.0 IP Connectivity (routing between VLANs using connected routes and default gateways).

## 📂 Files in This Lab

```
Inter-Vlan/
├── README.md
├── Inter-Vlan_Topology.JPG         # Topology diagram with addressing and VLAN labels
└── inter-Vlan_Configurations.txt   # Switch and router configuration commands
```

## 🚀 How to Open This Lab

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco NetAcad account).
2. Place one switch, one router, and two PCs. Connect them as shown in the Cabling table.
3. On SW1, open the CLI and paste the switch commands from the configuration file. On R1, paste the router commands.
4. Set each PC's address, mask and gateway in Desktop, then IP Configuration, using the PC table above.
5. On SW1, run `show vlan brief`. On R1, run `show ip route`. On PC0, run `ping 172.168.1.20`.

## 🔭 Possible Improvements

- **Replace the two physical links with a single trunk and router-on-a-stick.** Create subinterfaces `GigabitEthernet0/0.10` and `GigabitEthernet0/0.20` with `encapsulation dot1Q` to show the more scalable design.
- **Fix the port modes.** Add `switchport mode access` to Fa0/3 and Fa0/4 and confirm with `show interfaces status`.
- **Apply an access control list.** Block VLAN 20 from reaching VLAN 10 with an extended ACL on Gig0/1, then verify with `show access-lists`.
- **Use a private address for VLAN 20.** Change the network to 172.16.1.0/24 so it falls within RFC 1918, and update the table and PC settings.

## 👤 Author

[Your name] — Aspiring Network Engineer, building CCNA / networking fundamentals skills.

---
*This lab is part of my ongoing CCNA / Network Administrator study roadmap.*
