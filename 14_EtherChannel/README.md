[← Back to all labs](../README.md)

# EtherChannel with PAgP: Bundling Three Links Between Two Switches (Cisco Packet Tracer)

This lab bundles three physical links between two Catalyst switches into one logical link (a port-channel) using PAgP, Cisco's EtherChannel negotiation protocol. It shows how the bundle adds bandwidth and keeps traffic flowing if a member link fails, and how the `auto` and `desirable` modes decide whether the bundle forms.

## 📋 At a Glance

| | |
|---|---|
| **Lab number** | [N] |
| **Level** | Intermediate, CCNA |
| **Tools** | Cisco Packet Tracer, Cisco IOS CLI |
| **Devices** | 2 switches Catalyst 3560-24PS (SW-1, SW-2), 2 switches Catalyst 2960-24TT, 1 Server-PT (Server0), 2 PC-PT (PC0, PC1) |
| **Protocols / features** | EtherChannel, PAgP (auto and desirable modes), port-channel 1 |
| **Network(s)** | 192.168.1.0/24 (mask assumed, see Subnet Analysis) |
| **CCNA domains** | 2.0 Network Access |

## 📌 Project Objective

Bundle Fa0/2 to Fa0/4 on SW-1 and SW-2 into one port-channel using PAgP, so the two switches see the three links as a single logical link. Success means the port-channel comes up with all three members bundled, and that a PC on one side can reach the server on the other side over the bundle.

## 🌍 Real-World Scenario

Switches in a data centre or campus are often joined by several parallel cables. Without EtherChannel, spanning tree would block all but one of them to prevent loops, so the extra cables sit idle. EtherChannel combines them into one logical link that spanning tree treats as a single path. Traffic is spread across the members, and if one cable is cut the remaining members carry the load without any change to the rest of the network. Engineers use EtherChannel on uplinks between distribution and core switches, and on server connections where a single NIC would be a bottleneck.

## 🖧 Topology

![EtherChannel topology: SW-1 and SW-2 (3560-24PS) joined by three dashed links labelled EtherChannel, each with a 2960-24TT switch below and PCs and a server attached](Etherchannel_Lab.JPG)

SW-1 (Catalyst 3560-24PS, left) and SW-2 (Catalyst 3560-24PS, right) are joined by three parallel links, which the topology labels **EtherChannel**. Below SW-1 sits a 2960-24TT, labelled **SW**, that connects to **Server0** (192.168.1.3). Below SW-2 sits a second 2960-24TT, also labelled **SW**, that connects to **PC0** (192.168.1.1) and **PC1** (192.168.1.2). Both 2960s are labelled SW in the topology, so they are described here by the switch they sit under.

The topology does not show port numbers. The lab notes configure Fa0/2 to Fa0/4 on both 3560s, which is the range used for the bundle.

No routing is configured in the notes, so the lab runs at Layer 2. All hosts are in the same subnet and reach each other through the switches.

### Cabling

| From | Port | To | Port | Cable type |
|---|---|---|---|---|
| SW-1 | Fa0/2 | SW-2 | Fa0/2 | Crossover copper (switch to switch) |
| SW-1 | Fa0/3 | SW-2 | Fa0/3 | Crossover copper (switch to switch) |
| SW-1 | Fa0/4 | SW-2 | Fa0/4 | Crossover copper (switch to switch) |
| SW-1 | not recorded | 2960 (under SW-1) | not recorded | Crossover copper (switch to switch) |
| 2960 (under SW-1) | not recorded | Server0 | not recorded | Straight-through copper (switch to server) |
| SW-2 | not recorded | 2960 (under SW-2) | not recorded | Crossover copper (switch to switch) |
| 2960 (under SW-2) | not recorded | PC0 | not recorded | Straight-through copper (switch to PC) |
| 2960 (under SW-2) | not recorded | PC1 | not recorded | Straight-through copper (switch to PC) |

The port pairing for the three bundle links is assumed from the notes (Fa0/2 to Fa0/2, and so on). The cable types follow the standard rule. The solid lines in the topology join switches to PCs and the server, and the dashed lines join switches to each other. Packet Tracer's auto-MDIX can hide the difference, so the cable types are not confirmed by a screenshot.

### Addressing Table

| Device | Interface | IP Address | Subnet Mask | Prefix | Default Gateway |
|---|---|---|---|---|---|
| PC0 | NIC | 192.168.1.1 | not recorded | /24 (assumed) | not recorded |
| PC1 | NIC | 192.168.1.2 | not recorded | /24 (assumed) | not recorded |
| Server0 | NIC | 192.168.1.3 | not recorded | /24 (assumed) | not recorded |

The switches have no IP addresses in the notes, which is normal for a Layer 2 configuration.

### Subnet Analysis

| Network | Mask | Network Address | First Usable | Last Usable | Broadcast | Usable Hosts | Private (RFC 1918)? |
|---|---|---|---|---|---|---|---|
| 192.168.1.0/24 | 255.255.255.0 (assumed) | 192.168.1.0 | 192.168.1.1 | 192.168.1.254 | 192.168.1.255 | 254 | Yes (192.168.0.0/16) |

The notes give the three host addresses but not the mask. The addresses all sit in 192.168.1.0 to 192.168.1.255, so /24 is the most likely mask and is used here. Confirm it against the PC IP settings. Because all three hosts are in the same subnet, no default gateway is needed for this test.

## 📚 Concepts Behind This Lab

**Why bundle links.** EtherChannel combines up to 8 physical links into one logical link. Spanning tree sees the bundle as a single path, so it does not block the extra members. Load is shared across the members. The bundle has its own MAC address and its own interface, the port-channel. In this lab, three Fast Ethernet links give about 300 Mbps of aggregate capacity in each direction, but a single traffic flow uses only one member link, so one conversation is still limited to 100 Mbps.

**PAgP and LACP.** EtherChannel is negotiated by one of two protocols. PAgP is Cisco proprietary and is used in this lab. LACP (IEEE 802.3ad) is the open standard and works between non-Cisco devices.

| Protocol | Active / initiating mode | Passive / responding mode | Other mode |
|---|---|---|---|
| PAgP | `desirable` (initiates and replies) | (none) | `auto` (replies only) |
| LACP | `active` (initiates and replies) | `passive` (replies only) | (none) |
| Static | `on` (no negotiation) | `on` | `on` |

**Which mode pairs form a bundle.** The initiating side starts the negotiation, and a responding side answers it. A bundle forms when at least one end initiates.

| SW-1 mode | SW-2 mode | Bundle forms? |
|---|---|---|
| PAgP auto | PAgP desirable | Yes (used in this lab) |
| PAgP desirable | PAgP desirable | Yes |
| PAgP auto | PAgP auto | No (neither side initiates) |
| LACP active | LACP passive | Yes |
| LACP passive | LACP passive | No |

**Layer 2 requirements for members.** All member ports must match in speed, duplex, native VLAN, allowed VLANs and trunk or access mode. A mismatch can stop a port from bundling. The notes configure the members the same way on both switches, so these settings match.

**Load balancing.** Traffic is spread across members using a hash of addresses, usually the source and destination MAC addresses on Layer 2 switches. The hash is per flow, so a single flow always uses the same member. Check the method with `show etherchannel load-balance`.

## 🧭 Lab Steps

1. Connect the links as shown in the topology. Three parallel links join SW-1 and SW-2.
2. Assign IP addresses 192.168.1.1 (PC0), 192.168.1.2 (PC1) and 192.168.1.3 (Server0). These are from the notes.
3. On SW-1, configure Fa0/2 to Fa0/4 as a PAgP bundle in `auto` mode, which the notes call channel group 1.
4. On SW-2, configure Fa0/2 to Fa0/4 as a PAgP bundle in `desirable` mode, which also uses channel group 1.
5. Save both switches with `write`.
6. Verify the bundle with `show etherchannel summary` and `show interfaces port-channel 1`.

## ⚙️ Configuration

### Step 1: Configure the bundle on SW-1 (auto mode)

No screenshot was provided for this step. The commands below come from the lab notes.

```
enable
configure terminal
interface range FastEthernet0/2-4
 shutdown
 channel-group 1 mode auto
 no shutdown
end
write
```

| Command | Mode | What it does |
|---|---|---|
| `enable` | User EXEC | Enters privileged EXEC mode. |
| `configure terminal` | Privileged EXEC | Enters global configuration mode. |
| `interface range FastEthernet0/2-4` | Global config | Selects Fa0/2, Fa0/3 and Fa0/4 at once, so the same settings apply to all three. |
| `shutdown` | Interface range config | Disables the ports before changing them, which avoids a half-configured link. |
| `channel-group 1 mode auto` | Interface range config | Adds the ports to channel group 1 and sets PAgP to `auto`. The port-channel interface is created automatically. |
| `no shutdown` | Interface range config | Re-enables the ports so they can negotiate. |
| `end` | Interface config | Returns to privileged EXEC mode. |
| `write` | Privileged EXEC | Saves the running configuration to startup configuration. |

**Notes:** The notes put `shutdown` before the channel group and `no shutdown` after it. This is a good habit because the ports are not in use while their settings change. The `auto` mode waits for the other end to start the negotiation, so on its own it does not form a bundle. The `channel-group` number must match on both switches.

### Step 2: Configure the bundle on SW-2 (desirable mode)

No screenshot was provided for this step. The commands below come from the lab notes.

```
enable
configure terminal
interface range FastEthernet0/2-4
 shutdown
 channel-group 1 mode desirable
 no shutdown
end
write
```

| Command | Mode | What it does |
|---|---|---|
| `interface range FastEthernet0/2-4` | Global config | Selects the same three ports as on SW-1. |
| `shutdown` | Interface range config | Disables the ports while they are changed. |
| `channel-group 1 mode desirable` | Interface range config | Adds the ports to channel group 1 and sets PAgP to `desirable`, so this switch starts the negotiation. |
| `no shutdown` | Interface range config | Re-enables the ports. |
| `write` | Privileged EXEC | Saves the configuration. |

**Notes:** `desirable` initiates the negotiation and answers the other side. Because SW-2 starts it, the bundle forms even though SW-1 is in `auto`. If both sides were `auto`, neither would initiate and no bundle would form.

## ✅ Verification & Results

| Check | Command / Action | What success looks like | Result |
|---|---|---|---|
| Bundle summary on SW-1 | `show etherchannel summary` | Po1 shows `SU`, and Fa0/2, Fa0/3 and Fa0/4 show `P` | Expected (not captured) |
| Bundle summary on SW-2 | `show etherchannel summary` | Same as SW-1 | Expected (not captured) |
| Port-channel status | `show interfaces port-channel 1` | Port-channel 1 is up, line protocol is up | Expected (not captured) |
| Member ports | `show interfaces` | Each member shows connected and is part of the bundle | Expected (not captured) |
| Interface state | `show ip interface brief` | Interfaces listed with status up and protocol up for bundled ports | Expected (not captured) |
| PAgP neighbour | `show pagp neighbor` (additional command, not in the notes) | Neighbour on each member port is listed | Expected (not captured) |
| Traffic across the bundle | `ping 192.168.1.3` from PC0 | Replies from the server (the first ping may time out while ARP resolves) | Expected (not captured) |

**Expected output (not captured)**, `show etherchannel summary` on SW-1 after the bundle forms. Use it to check what to look for, not as a record of this run:

```
Flags:  D - down        P - bundled in port-channel
        I - stand-alone S - suspended
        R - Layer3      S - Layer2
        U - in use      N - not in use

Group  Port-channel  Protocol    Ports
------+-------------+-----------+------------------------------------------
1      Po1(SU)         PAgP      Fa0/2(P)    Fa0/3(P)    Fa0/4(P)
```

The `SU` flag on Po1 means the port-channel is in use at Layer 2. The `P` flag on each member means the port is bundled. A member showing `I` is stand-alone, which means it did not join the bundle.

## 🛠️ Troubleshooting Notes

**Problems the author hit:** None recorded in the notes or screenshots. No troubleshooting was noted in the lab material, so the table below lists the likely faults for this type of lab.

| Symptom | Likely cause | How to check | Fix |
|---|---|---|---|
| Ports show `I` (stand-alone), no bundle | Both ends in `auto`, or one end in `on` | `show etherchannel summary`, then the mode on each switch | Set one end to `desirable` (PAgP) or `active` (LACP) |
| Bundle forms with fewer than three members | A member port has different speed, duplex or VLAN settings | `show interfaces fa0/x switchport` on each member and compare | Make the settings identical on all members |
| Port-channel up, but traffic does not flow | Member mismatch on mode or channel number | Confirm the channel-group number matches on both switches | Re-run `channel-group` with matching numbers |
| Ports show `err-disabled` | A configuration problem such as a mismatch detected on a port | `show interfaces status err-disabled` | Fix the mismatch, then `shutdown` and `no shutdown` the port |
| Ping from PC0 to server times out | Routing or ARP, not the bundle (for example, a PC on a wrong subnet) | Check PC IP and mask, then `arp -a` on the PC | Correct the address, mask or cable |
| Traffic uses only one link | Normal hashing behaviour: one flow uses one member | `show etherchannel load-balance`, then test with several flows | Not a fault. Several flows are needed to see load shared |

## 💼 Skills Demonstrated

- Built a three-link EtherChannel between two Catalyst switches using PAgP.
- Used `interface range` to apply identical settings to multiple ports and reduce errors.
- Explained why `auto` and `desirable` form a bundle while `auto` and `auto` do not.
- Chose the shutdown, configure, no shutdown order to change member ports safely.
- Verified the bundle with `show etherchannel summary` and read its flags (SU, P, I).
- Explained how EtherChannel avoids spanning tree blocking and how traffic is hashed across members.

**CCNA 200-301 mapping:** 2.0 Network Access. Objective 2.4 (EtherChannel, Layer 2 and Layer 3). The lab also touches objective 2.2 (interswitch connectivity), because the bundle links switches together.

## 📂 Files in This Lab

```
EtherChannel/
├── README.md
├── Etherchannel_Lab.JPG            # topology diagram
└── EtherChannel_Configs_Notes.txt  # original configs and notes
```

## 🚀 How to Open This Lab

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco NetAcad account).
2. Build the topology from the **Topology** section, with three links between SW-1 and SW-2.
3. On SW-1 and SW-2, open the **CLI** tab and enter the commands from the **Configuration** section.
4. On SW-1, run `show etherchannel summary`. Po1 should show `SU` with three bundled members.
5. On PC0, open the **Command Prompt** and run `ping 192.168.1.3`.

## 🔭 Possible Improvements

- Test failover by shutting down one member link on SW-1 and checking that the bundle stays up with two members.
- Compare LACP (`active` and `passive`) on the same topology to see the difference between the two protocols.
- Configure a Layer 3 EtherChannel with `no switchport` and IP addresses on the port-channel, to show the routed version of the same bundle.

## 👤 Author

Anish Raj — Aspiring Network Engineer, building CCNA / networking fundamentals skills.

---
*This lab is part of my ongoing CCNA / Network Administrator study roadmap.*
