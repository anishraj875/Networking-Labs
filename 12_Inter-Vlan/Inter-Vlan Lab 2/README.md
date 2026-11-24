[← Back to all labs](../README.md)

# Inter-VLAN Routing with Router-on-a-Stick (Cisco Packet Tracer)

This lab connects a switch to a router with one trunk link and routes between two VLANs using subinterfaces on a single physical router port. It shows how one router interface can serve several VLANs, each with its own default gateway.

## 📋 At a Glance

| | |
|---|---|
| **Lab number** | [N] |
| **Level** | Beginner to Intermediate, CCNA |
| **Tools** | Cisco Packet Tracer, Cisco IOS CLI |
| **Devices** | 1 router (model not shown), 1 switch (model not shown), 2 PCs |
| **Protocols / features** | VLANs, 802.1Q trunking, router subinterfaces, access ports, IPv4 addressing, default gateway |
| **Network(s)** | 192.168.1.0/24 (VLAN 10, HR), 172.168.0.0/16 (VLAN 20, MR) |
| **CCNA domains** | Network Access (2.0), IP Connectivity (3.0), Network Fundamentals (1.0) |

## 📌 Project Objective

Create two VLANs on SW, place each PC in its VLAN, and connect the switch to the router with a single trunk link. Configure one subinterface on the router for each VLAN, with each subinterface using its VLAN's IP address as the default gateway. Success means PCs in different VLANs can reach each other through R1 over one physical link.

## 🌍 Real-World Scenario

Router-on-a-stick is common in small offices and branch sites that have a few VLANs and one router. Instead of one router port per VLAN, a single trunk carries all VLANs, and the router tags and routes traffic for each one. The design is simple and cheap, but all inter-VLAN traffic shares one link and the router's throughput, so larger networks move this job to a Layer 3 switch.

## 🖧 Topology

![Router-on-a-stick topology showing R1 connected to SW by a trunk on Fa0/1, with PC0 (VLAN 10, HR) on Fa0/2 and PC1 (VLAN 20, MR) on Fa0/3](Router-on-Stick_.JPG)

The router R1 connects to the switch SW on one link: R1 Gi0/0 to SW Fa0/1, which is configured as a trunk. PC0 (VLAN 10, HR) connects to Fa0/2 and PC1 (VLAN 20, MR) connects to Fa0/3. The diagram labels R1 Gi0/0.1 as 192.168.1.1 for VLAN 10 and Gi0/0.2 as 172.168.1.1 for VLAN 20.

### Cabling

| From | Port | To | Port | Cable type |
|---|---|---|---|---|
| SW | Fa0/1 | R1 | Gig0/0 | Straight-through (copper) |
| SW | Fa0/2 | PC0 | Fa0 | Straight-through (copper) |
| SW | Fa0/3 | PC1 | Fa0 | Straight-through (copper) |

The cable type is not labeled on the diagram, so these follow the standard rule: a switch and a PC, or a switch and a router, are unlike devices and use straight-through cable.

### Addressing Table

| Device | Interface | IP Address | Subnet Mask | Prefix | Default Gateway |
|---|---|---|---|---|---|
| R1 | GigabitEthernet0/0.1 (Gi0/0.1, VLAN 10) | 192.168.1.1 | 255.255.255.0 | /24 | Not applicable (router interface) |
| R1 | GigabitEthernet0/0.2 (Gi0/0.2, VLAN 20) | 172.168.1.1 | 255.255.0.0 | /16 | Not applicable (router interface) |
| PC0 (HR) | Fa0 | 192.168.1.10 | 255.255.255.0 | /24 | 192.168.1.1 |
| PC1 (MR) | Fa0 | 172.168.1.10 | 255.255.0.0 | /16 | 172.168.1.1 |

**Note on PC0:** the configuration file lists PC0 as 192.168.1.10, but the topology label reads 192.168.10.1. The label looks like a typo, since 192.168.10.1 is outside the 192.168.1.0/24 subnet that VLAN 10 uses. This README uses 192.168.1.10 from the configuration file. Confirm which address is set in the Packet Tracer file.

### Subnet Analysis

| Network | Mask | Network Address | First Usable | Last Usable | Broadcast | Usable Hosts | Private (RFC 1918)? |
|---|---|---|---|---|---|---|---|
| VLAN 10 (HR) | 255.255.255.0 (/24) | 192.168.1.0 | 192.168.1.1 | 192.168.1.254 | 192.168.1.255 | 254 | Yes (192.168.0.0/16) |
| VLAN 20 (MR) | 255.255.0.0 (/16) | 172.168.0.0 | 172.168.0.1 | 172.168.255.254 | 172.168.255.255 | 65,534 | No (see note below) |

Each VLAN has its own subnet. The router subinterface on that VLAN holds the first usable address (.1), which is the default gateway for the VLAN's PCs. Both gateways sit inside their own subnets.

**Address range note:** 172.168.0.0/16 is not a private range. RFC 1918 reserves 172.16.0.0 to 172.31.255.255 (172.16.0.0/12), and 172.168 falls outside that block. In a real network this range would be publicly routable, and 172.16.1.0/24 would follow the standard. The same note applies to the previous inter-VLAN lab.

## 📚 Concepts Behind This Lab

**Router-on-a-stick.** One physical router interface carries traffic for several VLANs. The switch port that connects to the router is a trunk, so frames from every VLAN cross the link with an 802.1Q tag that says which VLAN they belong to. The router creates one logical subinterface for each VLAN. Each subinterface handles only the frames tagged for its VLAN and acts as that VLAN's gateway.

**802.1Q trunking.** The trunk adds a 4-byte tag to each frame to identify its VLAN. The native VLAN (default VLAN 1) is sent untagged, and both ends of the trunk must agree on it. In this lab the router does not send untagged traffic, so the default native VLAN causes no problem. Set `switchport mode trunk` on the switch port explicitly. A router does not negotiate trunks with DTP, so leaving the port in dynamic auto mode can leave it as an access port and stop the VLANs from crossing the link.

**Subinterfaces.** The subinterface number is a label and does not have to match the VLAN number, but numbering it to match the VLAN (Gi0/0.10 for VLAN 10) is clearer. The number in `encapsulation dot1Q` is the VLAN number that matters. The encapsulation line must come before the IP address line, and the parent interface Gi0/0 needs `no shutdown` for any subinterface to work.

**Access ports for the PCs.** Each PC connects to an access port that belongs to one VLAN and carries untagged frames. The PC does not know about VLAN tags, so it needs nothing special. Access ports are used for the PC links, and the trunk is used only for the router link.

**Default gateway.** A PC sends traffic for any address outside its own subnet to its default gateway. PC0 sends traffic for 172.168.0.0/16 to 192.168.1.1, and PC1 sends traffic for 192.168.1.0/24 to 172.168.1.1. Each gateway must be on the subinterface that matches the PC's VLAN.

**Comparison with one interface per VLAN.** The previous lab used a separate physical router link for each VLAN. This design needs one link and one switch trunk port. The trade-off is that all inter-VLAN traffic shares a single link, so the link and the router become a bottleneck as traffic grows.

| | One router interface per VLAN | Router-on-a-stick (this lab) |
|---|---|---|
| Physical links to the router | One per VLAN | One trunk for all VLANs |
| Switch port to router | Access port | Trunk port |
| Router configuration | IP address on each physical interface | Subinterface per VLAN with `encapsulation dot1Q` |
| Bandwidth between VLANs | A separate link per VLAN | Shared single link |
| Scales to many VLANs? | Poorly, because it needs many router ports | Better, but the single link limits throughput |

## 🧭 Lab Steps

1. Build the topology: one switch, one router, two PCs, and three straight-through cables as listed in the Cabling table.
2. On SW, create VLAN 10 (named HR) and VLAN 20 (named MR).
3. On SW, assign Fa0/2 to VLAN 10 and Fa0/3 to VLAN 20 as access ports for the PCs.
4. On SW, set Fa0/1 to trunk mode for the router link.
5. On R1, bring up the physical interface Gi0/0 with `no shutdown`.
6. On R1, create subinterface Gi0/0.1 for VLAN 10 and Gi0/0.2 for VLAN 20, with the encapsulation and IP address for each.
7. Set each PC's address, mask and default gateway from the Addressing Table.
8. Verify the trunk, VLAN membership, the router's subinterfaces and routing table, and pings between the two VLANs.

## ⚙️ Configuration

### Step 1: Create the VLANs and configure the switch ports

```
enable
configure terminal
hostname SW
vlan 10
 name HR
vlan 20
 name MR
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 10
interface fastEthernet 0/3
 switchport mode access
 switchport access vlan 20
interface fastEthernet 0/1
 switchport mode trunk
end
```

| Command | Mode | What it does |
|---|---|---|
| `enable` | User EXEC | Enters privileged EXEC mode (`SW#`) |
| `configure terminal` | Privileged EXEC | Enters global configuration mode (`SW(config)#`) |
| `hostname SW` | Global config | Sets the switch name shown in the prompt |
| `vlan 10` / `name HR` | Global config / VLAN config | Creates and names VLAN 10 |
| `vlan 20` / `name MR` | Global config / VLAN config | Creates and names VLAN 20 |
| `interface fastEthernet 0/2` | Global config | Enters configuration for port Fa0/2 (`SW(config-if)#`) |
| `switchport mode access` | Interface config | Makes the port a single-VLAN access port |
| `switchport access vlan 10` | Interface config | Places Fa0/2 in VLAN 10 |
| `interface fastEthernet 0/3` | Global config | Enters configuration for port Fa0/3 |
| `switchport access vlan 20` | Interface config | Places Fa0/3 in VLAN 20 |
| `interface fastEthernet 0/1` | Global config | Enters configuration for the router link |
| `switchport mode trunk` | Interface config | Makes Fa0/1 a trunk that carries tagged frames for all VLANs |
| `end` | Any config mode | Returns to privileged EXEC mode |

**Notes:**
- The configuration file uses `host SW`. The IOS command is `hostname SW`, so the tidied version uses `hostname`.
- Fa0/1 is set to `switchport mode trunk` explicitly. Routers do not run DTP, so the trunk must be forced on the switch side.
- No `switchport trunk allowed vlan` line was configured, so the trunk carries all VLANs by default. Restricting it to `10,20` is a good hardening step.

### Step 2: Configure the router subinterfaces

```
enable
configure terminal
hostname R1
interface GigabitEthernet0/0
 no shutdown
interface GigabitEthernet0/0.1
 encapsulation dot1Q 10
 ip address 192.168.1.1 255.255.255.0
interface GigabitEthernet0/0.2
 encapsulation dot1Q 20
 ip address 172.168.1.1 255.255.0.0
end
```

| Command | Mode | What it does |
|---|---|---|
| `hostname R1` | Global config | Names the router |
| `interface GigabitEthernet0/0` | Global config | Enters configuration for the physical link to SW Fa0/1 |
| `no shutdown` | Interface config | Brings up the physical interface; router interfaces start administratively down |
| `interface GigabitEthernet0/0.1` | Global config | Creates or enters subinterface Gi0/0.1 (`R1(config-subif)#`) |
| `encapsulation dot1Q 10` | Subinterface config | Tags traffic on this subinterface with VLAN 10 |
| `ip address 192.168.1.1 255.255.255.0` | Subinterface config | Gives this subinterface the VLAN 10 gateway address |
| `interface GigabitEthernet0/0.2` | Global config | Creates or enters subinterface Gi0/0.2 |
| `encapsulation dot1Q 20` | Subinterface config | Tags traffic on this subinterface with VLAN 20 |
| `ip address 172.168.1.1 255.255.0.0` | Subinterface config | Gives this subinterface the VLAN 20 gateway address |

**Notes:**
- The configuration file uses `host R1` and `encapsulation dot1q`. The tidied version uses `hostname` and `dot1Q`. IOS accepts either letter case.
- Put `encapsulation` before `ip address`. A subinterface must know its VLAN tag before it can hold an address.
- If the parent interface is shut down, both subinterfaces go down with it. Confirm the parent with `show ip interface brief`.
- The file's `int g0/0.1` and `int g0/0.2` use the Gi0/0 subinterface names. These are the same subinterfaces as the GigabitEthernet names above.

### Step 3: Configure the PCs

The file gives the values for each PC. Set them in each PC's Desktop, under IP Configuration.

| PC | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC0 (HR) | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC1 (MR) | 172.168.1.10 | 255.255.0.0 | 172.168.1.1 |

**Notes:** The default gateway must be the subinterface for the PC's own VLAN. PC0 uses Gi0/0.1 and PC1 uses Gi0/0.2. Both PCs use the same physical router link, and the subinterface numbers keep their traffic separate.

## ✅ Verification & Results

| Check | Command / Action | What success looks like | Result |
|---|---|---|---|
| Trunk formed | `show interfaces trunk` on SW | Fa0/1 shows `trunking`, with VLANs 10 and 20 allowed | Expected (not captured) |
| VLAN membership | `show vlan brief` on SW | VLAN 10 lists Fa0/2; VLAN 20 lists Fa0/3; Fa0/1 is not in an access VLAN | Expected (not captured) |
| Router interface state | `show ip interface brief` on R1 | Gi0/0, Gi0/0.1 and Gi0/0.2 show up/up, with the addresses above | Expected (not captured) |
| Subinterface tagging | `show vlan` or `show interfaces GigabitEthernet0/0.1` on R1 | Gi0/0.1 shows `802.1Q Virtual LAN, Vlan ID 10` | Expected (not captured) |
| Routing table | `show ip route` on R1 | Two `C` (connected) entries, one per subinterface | Expected (not captured) |
| PC settings | `ipconfig` on PC0 and PC1 | Address, mask and gateway match the Addressing Table | Expected (not captured) |
| Gateway reachability | `ping 192.168.1.1` from PC0; `ping 172.168.1.1` from PC1 | Replies from each PC's own gateway | Expected (not captured) |
| Inter-VLAN ping | `ping 172.168.1.10` from PC0; `ping 192.168.1.10` from PC1 | Replies in both directions, through R1 | Expected (not captured) |
| Path check | `tracert 172.168.1.10` from PC0 | Hop 1 is 192.168.1.1 (R1), then the destination | Expected (not captured) |

No screenshots of command output were provided, so every result above is expected and not captured.

**Expected output: `show interfaces trunk` on SW**

```
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/1       1-1005
```

**Expected output: `show ip route` on R1**

```
C    172.168.0.0/16 is directly connected, GigabitEthernet0/0.2
C    192.168.1.0/24 is directly connected, GigabitEthernet0/0.1
```

Packet Tracer often shows a timeout on the first ping while ARP resolves. Later pings succeed. A TTL of 127 on a reply means the packet crossed one router hop, which confirms the routing happened through R1 rather than the switch.

## 🛠️ Troubleshooting Notes

No problems were recorded in the files provided. The table below lists the most likely failures for this lab and how to check each one, working from Layer 1 upward.

| Symptom | Likely cause | How to check | Fix |
|---|---|---|---|
| Subinterfaces are down or missing | Parent Gi0/0 shut down | `show ip interface brief` on R1 | Enter `interface GigabitEthernet0/0` and run `no shutdown` |
| VLANs do not cross the link; PC-to-PC ping fails | Fa0/1 not in trunk mode | `show interfaces trunk` on SW | Set `switchport mode trunk` on Fa0/1 |
| Inter-VLAN ping fails with a correct gateway | Wrong `encapsulation dot1Q` number on a subinterface | `show run interface GigabitEthernet0/0.1` | Correct the VLAN number to match the subinterface's VLAN |
| Subinterface has no VLAN tag | `encapsulation` line missing or entered after `ip address` | `show interfaces GigabitEthernet0/0.1` | Enter `encapsulation dot1Q` on the subinterface |
| A PC cannot reach its gateway | PC plugged into the wrong switch port | `show vlan brief` on SW | Move the cable to the access port for the PC's VLAN |
| PC reaches its own VLAN but not the other | Wrong default gateway on the PC | `ipconfig` on the PC | Set the gateway to the subinterface for the PC's VLAN |
| Trunk forms but traffic is missing for one VLAN | Trunk `allowed vlan` list excludes the VLAN | `show interfaces trunk` on SW | Add the VLAN with `switchport trunk allowed vlan add 20` |

**General tip:** Check Layer 1 and Layer 2 before Layer 3. A trunk that is not formed prevents every inter-VLAN ping, so confirm `trunking` on the switch before changing the router's IP addresses.

## 💼 Skills Demonstrated

- Configured an 802.1Q trunk on a switch port and verified it with `show interfaces trunk`.
- Created router subinterfaces, each tagged with a VLAN and configured as that VLAN's default gateway.
- Planned and verified IPv4 addressing for two subnets with different masks (/24 and /16).
- Explained how one physical link carries traffic for multiple VLANs and why the router is needed to route between them.
- Compared router-on-a-stick with a one-interface-per-VLAN design and identified the trade-offs.

**CCNA 200-301 mapping:** 2.0 Network Access (VLAN configuration, 802.1Q trunking, access ports); 1.0 Network Fundamentals (IPv4 addressing and subnetting); 3.0 IP Connectivity (inter-VLAN routing using subinterfaces).

## 📂 Files in This Lab

```
Router-on-Stick/
├── README.md
├── Router-on-Stick_.JPG                 # Topology diagram with addressing and VLAN labels
└── Router-on-Stick_Configuration.txt    # Switch and router configuration commands
```

## 🚀 How to Open This Lab

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco NetAcad account).
2. Place one switch, one router, and two PCs. Connect them as shown in the Cabling table.
3. On the switch CLI, paste the switch commands from the configuration file. On the router CLI, paste the router commands.
4. Set each PC's address, mask and gateway in Desktop, then IP Configuration, using the PC table above.
5. On SW, run `show interfaces trunk`. On R1, run `show ip route`. On PC0, run `ping 172.168.1.10`.

## 🔭 Possible Improvements

- **Restrict the trunk to the VLANs in use.** Run `switchport trunk allowed vlan 10,20` on Fa0/1 and confirm with `show interfaces trunk`.
- **Add a native VLAN that is not used.** Set `switchport trunk native vlan 99` on both ends of the trunk, which keeps VLAN 1 out of the trunk.
- **Apply an access control list.** Block VLAN 20 from reaching VLAN 10 with an extended ACL on Gi0/0.1, then verify with `show access-lists`.
- **Move to a private address for VLAN 20.** Change the network to 172.16.1.0/24, then update the table and PC settings.

## 👤 Author

[Your name] — Aspiring Network Engineer, building CCNA / networking fundamentals skills.

---
*This lab is part of my ongoing CCNA / Network Administrator study roadmap.*
