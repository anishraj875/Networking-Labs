[← Back to all labs](../README.md)

# VLAN Configuration: Creating VLANs and Assigning Switch Ports (Cisco Packet Tracer)

This lab segments a single switch into four VLANs (HR, MR, CO and RE) and assigns two access ports to each one. It shows how VLANs split one physical switch into separate broadcast domains.

## 📋 At a Glance

| | |
|---|---|
| **Lab number** | _(add your lab number)_ |
| **Level** | Beginner, CCNA |
| **Tools** | Cisco Packet Tracer, Cisco IOS CLI |
| **Devices** | 1 switch (renamed `SW1`), 8 PCs |
| **Protocols / features** | VLANs (802.1Q VLAN database), access ports, `show vlan` verification |
| **Network(s)** | 192.168.10.0/24 (mask assumed, see Subnet Analysis) |
| **CCNA domains** | Network Fundamentals, Network Access |

## 📌 Project Objective

Create four VLANs on one switch, give each a name, and place two PCs in each VLAN by assigning switch ports. The result is confirmed with `show vlan`, which lists every VLAN with its member ports.

| VLAN ID | Name | Department (as labelled in topology) | Ports | PCs |
|---|---|---|---|---|
| 2 | HR | HR | Fa0/1 – Fa0/2 | 192.168.10.1, 192.168.10.2 |
| 3 | MR | MR | Fa0/3 – Fa0/4 | 192.168.10.3, 192.168.10.4 |
| 4 | CO | CO | Fa0/5 – Fa0/6 | 192.168.10.5, 192.168.10.6 |
| 5 | RE | RE | Fa0/7 – Fa0/8 | 192.168.10.7, 192.168.10.8 |

## 🌍 Real-World Scenario

_Background context, not something done in this lab._ Companies place departments such as HR, finance or engineering in separate VLANs so that a broadcast from one department does not reach the others and so that sensitive systems can be isolated. One physical switch can serve all of them, which saves hardware compared with buying a switch per department.

## 🖧 Topology

![Network topology showing one switch connected to eight PCs grouped into four VLANs: HR VLAN-2, MR VLAN-3, CO VLAN-4 and RE VLAN-5](VLan_Topology.JPG)

*The topology image shows one central switch with eight PCs. Each pair is drawn in a box labelled with the department, VLAN number and switch ports: HR VLAN-2 on Fa0/1 - 0/2, MR VLAN-3 on Fa0/3 - 0/4, CO VLAN-4 on Fa0/5 - 0/6 and RE VLAN-5 on Fa0/7 - 0/8.*

### Cabling

Each PC uses its `Fa0` NIC. The port-to-PC mapping follows the box labels in the topology (two ports per department). Which individual PC sits on which of the two ports is not legible in the image, so it is not listed.

| From | Port | To | Port | Cable type |
|---|---|---|---|---|
| PC (192.168.10.1, .2) | Fa0 | SW1 | Fa0/1, Fa0/2 | Straight-through (standard rule for PC to switch) |
| PC (192.168.10.3, .4) | Fa0 | SW1 | Fa0/3, Fa0/4 | Straight-through |
| PC (192.168.10.5, .6) | Fa0 | SW1 | Fa0/5, Fa0/6 | Straight-through |
| PC (192.168.10.7, .8) | Fa0 | SW1 | Fa0/7, Fa0/8 | Straight-through |

The cable type is not visible in the image; straight-through is the standard choice for connecting an end device to a switch. The topology shows green link lights on all eight links.

### Addressing Table

The mask and gateway are not shown in the supplied material. The table gives the IPs from the topology and `Commands.txt`.

| Device | Interface | IP Address | Subnet Mask | VLAN | Default Gateway |
|---|---|---|---|---|---|
| PC (HR) | Fa0 | 192.168.10.1 | Not shown (/24 assumed) | 2 | None shown |
| PC (HR) | Fa0 | 192.168.10.2 | Not shown (/24 assumed) | 2 | None shown |
| PC (MR) | Fa0 | 192.168.10.3 | Not shown (/24 assumed) | 3 | None shown |
| PC (MR) | Fa0 | 192.168.10.4 | Not shown (/24 assumed) | 3 | None shown |
| PC (CO) | Fa0 | 192.168.10.5 | Not shown (/24 assumed) | 4 | None shown |
| PC (CO) | Fa0 | 192.168.10.6 | Not shown (/24 assumed) | 4 | None shown |
| PC (RE) | Fa0 | 192.168.10.7 | Not shown (/24 assumed) | 5 | None shown |
| PC (RE) | Fa0 | 192.168.10.8 | Not shown (/24 assumed) | 5 | None shown |

### Subnet Analysis

_Assuming the Packet Tracer default Class C mask 255.255.255.0 (/24)._

| Network | Mask | Network Address | First Usable | Last Usable | Broadcast | Usable Hosts | Private (RFC 1918)? |
|---|---|---|---|---|---|---|---|
| 192.168.10.0/24 | 255.255.255.0 | 192.168.10.0 | 192.168.10.1 | 192.168.10.254 | 192.168.10.255 | 254 | Yes (192.168.0.0/16) |

**Design observation:** all eight PCs share one IP subnet, while the switch places them in four different VLANs. A VLAN is a separate broadcast domain, so ARP broadcasts do not cross VLANs. As a result, PCs in the same VLAN should reach each other, while PCs in different VLANs should not, even though their IPs look like they belong together. In a production design each VLAN would normally get its own subnet, with a router or Layer 3 switch routing between them (see Possible Improvements).

## 📚 Concepts Behind This Lab

**VLAN (Virtual LAN).** A VLAN is a logical broadcast domain created on a switch. Devices in the same VLAN can exchange Layer 2 traffic; devices in different VLANs cannot, without a Layer 3 device. This improves security, reduces broadcast traffic and lets the network follow the organisation's structure rather than the physical layout.

**VLAN ranges.** The normal range is 1 to 1005 and the extended range is 1006 to 4094. VLAN 1 is the default, and every port starts in it. In this lab the VLANs 2 to 5 are in the normal range, so they are stored in the VLAN database. The show output also lists the reserved legacy VLANs 1002 to 1005 (fddi, token-ring), which exist by default and cannot be deleted.

**Access port.** An access port belongs to exactly one VLAN and carries untagged traffic for an end device such as a PC. `switchport access vlan N` moves the port into VLAN N.

**Broadcast and collision domains.** Each switch port is its own collision domain, so the eight connected ports are eight collision domains. Each VLAN is its own broadcast domain, giving four broadcast domains for the VLANs in use (VLAN 1 still exists but holds no connected devices).

| | Without VLANs | With VLANs (this lab) |
|---|---|---|
| Broadcast domains on the switch | 1 | 4 (for the connected ports) |
| Can HR reach MR at Layer 2? | Yes | No |
| Device needed to connect the groups | None | Router or Layer 3 switch |

## 🧭 Lab Steps

1. Build the topology: one switch and eight PCs, two PCs per department (HR, MR, CO, RE).
2. Set the PC IP addresses 192.168.10.1 to 192.168.10.8 as shown in the topology.
3. On the switch, enter privileged and global configuration mode and rename it `SW1`.
4. Create VLAN 2 (HR), VLAN 3 (MR), VLAN 4 (CO) and VLAN 5 (RE).
5. Assign Fa0/1-2 to VLAN 2, Fa0/3-4 to VLAN 3, Fa0/5-6 to VLAN 4 and Fa0/7-8 to VLAN 5.
6. Verify the VLANs and port membership with `show vlan`.

## ⚙️ Configuration

### Step 1: Rename the switch and create the VLANs

![Terminal screenshot showing hostname SW1 and VLANs 2 to 5 created with names HR, MR, CO and RE](Creating_Vlan.JPG)

*Screenshot: hostname set to SW1, then VLANs 2 to 5 created and named.*

```
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# vlan 2
SW1(config-vlan)# name HR
SW1(config-vlan)# vlan 3
SW1(config-vlan)# name MR
SW1(config-vlan)# vlan 4
SW1(config-vlan)# name CO
SW1(config-vlan)# vlan 5
SW1(config-vlan)# name RE
SW1(config-vlan)# exit
```

| Command | Mode | What it does |
|---|---|---|
| `enable` (typed `en`) | User EXEC | Enters privileged EXEC mode (prompt changes from `>` to `#`). |
| `configure terminal` (typed `conf t`) | Privileged EXEC | Enters global configuration mode. |
| `hostname SW1` | Global config | Renames the device; the prompt changes from `Switch(config)#` to `SW1(config)#`. |
| `vlan 2` | Global config | Creates VLAN 2 (or opens it if it exists) and enters VLAN config mode (`SW1(config-vlan)#`). |
| `name HR` | VLAN config | Gives VLAN 2 a descriptive name. The name does not change how the VLAN works. |
| `vlan 3`, `vlan 4`, `vlan 5` | VLAN config / Global config | Creates the next VLANs. Typing `vlan N` from VLAN config mode works the same as from global config. |
| `exit` | VLAN config | Returns to global configuration mode. |

**Notes:** On a 2960-style switch the VLAN is created as soon as you leave VLAN config mode. VLAN names are optional but make `show vlan` far easier to read.

### Step 2: Assign switch ports to the VLANs

![Terminal screenshot showing interface range commands assigning Fa0/1-2 to VLAN 2, Fa0/3-4 to VLAN 3, Fa0/5-6 to VLAN 4 and Fa0/7-8 to VLAN 5](Assigning_Ports.JPG)

*Screenshot: each pair of ports is placed into its VLAN with `interface range` and `switchport access vlan`.*

```
SW1(config)# interface range fastEthernet 0/1-2
SW1(config-if-range)# switchport access vlan 2
SW1(config-if-range)# exit
SW1(config)# interface range fastEthernet 0/3-4
SW1(config-if-range)# switchport access vlan 3
SW1(config-if-range)# exit
SW1(config)# interface range fastEthernet 0/5-6
SW1(config-if-range)# switchport access vlan 4
SW1(config-if-range)# exit
SW1(config)# interface range fastEthernet 0/7-8
SW1(config-if-range)# switchport access vlan 5
```

| Command | Mode | What it does |
|---|---|---|
| `interface range fastEthernet 0/1-2` (typed `int range fa 0/1-2`) | Global config | Selects several interfaces at once so one set of commands applies to all of them. Prompt becomes `SW1(config-if-range)#`. |
| `switchport access vlan 2` | Interface range | Puts the selected ports into VLAN 2 as access ports. |
| `exit` | Interface range | Returns to global configuration mode. |

**Notes:** The author's transcript shows some partially typed commands (`switchpor`, `acces`, and an incomplete `int range fa`) that were corrected or retyped; only the completed commands are listed above. VLAN membership only works if the VLAN exists, so Step 1 comes before Step 2 (on a real switch, assigning a port to a missing VLAN creates it automatically, but it would then have no name). The configuration was not saved to NVRAM in the supplied material; `copy running-config startup-config` would be needed to keep it after a reload.

## ✅ Verification & Results

| Check | Command / Action | What success looks like | Result |
|---|---|---|---|
| VLANs exist with correct names | `show vlan` | VLANs 2 to 5 appear as `HR`, `MR`, `CO`, `RE` with status `active` | Shown in screenshot |
| Ports are in the right VLANs | `show vlan` | Fa0/1-2 in VLAN 2, Fa0/3-4 in VLAN 3, Fa0/5-6 in VLAN 4, Fa0/7-8 in VLAN 5 | Shown in screenshot |
| Unused ports stay in VLAN 1 | `show vlan` | Fa0/9 to Fa0/24, Gig0/1 and Gig0/2 listed under `default` | Shown in screenshot |
| Same-VLAN connectivity | `ping` between the two PCs of one VLAN (for example 192.168.10.1 to 192.168.10.2) | Replies received (the first ping may time out while ARP resolves) | Expected (not captured) |
| VLAN isolation | `ping` from an HR PC to an MR PC (for example 192.168.10.1 to 192.168.10.3) | Request timed out, because VLANs are separate broadcast domains and no router is present | Expected (not captured) |

![Terminal screenshot of the show vlan output listing VLAN 1 default, VLAN 2 HR, VLAN 3 MR, VLAN 4 CO and VLAN 5 RE with their ports](Show_Vlan.JPG)

*Screenshot: `show vlan` output confirming the VLAN names and port membership.*

Captured output (from `Commands.txt`; the screenshot shows the same table, cut off after VLAN 3 in the second block):

```
SW1# show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                Gig0/1, Gig0/2
2    HR                               active    Fa0/1, Fa0/2
3    MR                               active    Fa0/3, Fa0/4
4    CO                               active    Fa0/5, Fa0/6
5    RE                               active    Fa0/7, Fa0/8
1002 fddi-default                     active
1003 token-ring-default               active
1004 fddinet-default                  active
1005 trnet-default                    active
```

**How to read it:** the `Ports` column lists access ports only. Trunk ports do not appear in `show vlan`; they show in `show interfaces trunk`. The switch has 24 FastEthernet ports and two Gigabit ports, which matches a Catalyst 2960-24 style model, but the exact model is not stated in the supplied material.

## 🛠️ Troubleshooting Notes

_General tips for this lab type, not problems reported by the author._

| Symptom | Likely cause | How to check | Fix |
|---|---|---|---|
| A PC cannot ping a PC in the same VLAN | Port is in the wrong VLAN or in VLAN 1 | `show vlan` and look at the Ports column | Re-run `switchport access vlan N` on the correct port |
| Port shows in the wrong VLAN after typing the command | Command applied to the wrong interface range | `show running-config` and review the `interface` blocks | Reapply the command to the right range |
| Ping between different VLANs fails | Expected behaviour: no Layer 3 device between VLANs | Confirm there is no router or SVI | Add router-on-a-stick or a Layer 3 switch if inter-VLAN traffic is wanted |
| VLAN missing after switch reload | Configuration not saved | `show startup-config` | `copy running-config startup-config` |
| Link light is red or amber | Cable or port problem | `show interfaces status` | Check cable type and that the port is not shut down |
| First ping times out, then succeeds | ARP resolving (normal) | Ping again | None needed |

## 💼 Skills Demonstrated

- Created and named VLANs from the IOS command line.
- Assigned access ports to VLANs using `interface range` to apply configuration to several ports at once.
- Verified VLAN and port membership with `show vlan` and interpreted the output.
- Documented a topology with VLAN IDs, ports and IP addresses.
- Explained how VLANs create separate broadcast domains and why inter-VLAN traffic needs Layer 3.

**CCNA 200-301 mapping:** 2.0 Network Access (2.1 Configure and verify VLANs, normal range, spanning multiple switches; the multi-switch and trunking parts are not covered here); 1.0 Network Fundamentals (1.1 Network components, 1.3 Network topology architectures, interface and cable connectivity).

## 📂 Files in This Lab

Layout assumed to be a flat folder, matching the files supplied. Adjust the image links if you use subfolders.

```
[Lab folder name]/
├── README.md
├── VLan_Topology.JPG       # Topology screenshot
├── Creating_Vlan.JPG       # VLAN creation commands
├── Assigning_Ports.JPG     # Port assignment commands
├── Show_Vlan.JPG           # show vlan output
├── Commands.txt            # Typed commands and show vlan output
└── Vlan.pkt                # Packet Tracer lab file
```

## 🚀 How to Open This Lab

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco NetAcad account).
2. Open `Vlan.pkt`.
3. Click the switch, open the **CLI** tab and run `enable` followed by `show vlan`.
4. To test connectivity, click a PC, open **Desktop > Command Prompt** and try `ping 192.168.10.2` (same VLAN) and `ping 192.168.10.3` (different VLAN).

## 🔭 Possible Improvements

- Give each VLAN its own subnet (for example 192.168.20.0/24 for VLAN 2) and set a default gateway on each PC.
- Add a router and configure router-on-a-stick (a trunk with 802.1Q subinterfaces) to allow inter-VLAN routing.
- Add a second switch and a trunk link to extend the VLANs across both switches.
- Set `switchport mode access` explicitly on the access ports and shut down unused ports for better security.

## 👤 Author

Anish Raj — Aspiring Network Engineer, building CCNA / networking fundamentals skills.

---
*This lab is part of my ongoing CCNA / Network Administrator study roadmap.*
