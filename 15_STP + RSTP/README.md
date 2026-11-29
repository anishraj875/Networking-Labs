[← Back to all labs](../README.md)

# STP and RSTP Lab: Root Bridge Election and a Blocked Redundant Link (Cisco Packet Tracer)

This lab uses three switches joined in a triangle to show Spanning Tree Protocol (STP) at work. The switches elect a root bridge, each non-root switch picks one root port, and one redundant link is blocked to prevent a Layer 2 loop. The notes also cover RSTP, PortFast and BPDU Guard.

## 📋 At a Glance

| | |
|---|---|
| **Lab number** | [N] |
| **Level** | Intermediate, CCNA |
| **Tools** | Cisco Packet Tracer, Cisco IOS CLI |
| **Devices** | 3 switches (SW-1 Root Bridge, SW-2, SW-3), 2 PC-PT (PC2, PC3). Switch model not recorded. |
| **Protocols / features** | STP (IEEE 802.1D), root bridge election, port roles and states, RSTP (rapid-pvst), PortFast, BPDU Guard |
| **Network(s)** | 10.0.0.0/24 (mask assumed, see Subnet Analysis) |
| **CCNA domains** | 2.0 Network Access |

## 📌 Project Objective

Show that a triangle of switches forms a loop, let STP elect a root bridge, and confirm that exactly one redundant link is blocked. Success means SW-1 is the root bridge, SW-2 and SW-3 each have one root port, and SW-2's link to SW-3 is blocked, with PC2 and PC3 still able to reach each other over the remaining path.

## 🌍 Real-World Scenario

Engineers add redundant links between switches so that one cable failure does not cut off a whole floor or building. Without STP, that redundancy creates a loop. Broadcasts keep circulating, the MAC address table flaps, and the network can freeze within seconds. STP keeps the redundant links in place but blocks the ones that would form a loop, and it unblocks them automatically if the active path fails. Modern networks use RSTP, which recovers in seconds rather than the tens of seconds that classic STP needs.

## 🖧 Topology

![STP topology: SW-1 marked Root Bridge with priority 4097, SW-2 and SW-3 at priority 32769, PC2 at 10.0.0.10 and PC3 at 10.0.0.20, and an orange marker on the SW-2 end of the SW-2 to SW-3 link](STP___RSTP.JPG)

The topology shows three switches in a triangle. SW-1 is labelled **Root Bridge** and shows bridge ID **4097** with MAC address **00E0.F9C0.3DAB**. SW-2 and SW-3 each show **32769** (the default priority plus the VLAN 1 system ID), with MAC addresses **00E0.B08D.B7CA** (SW-2) and **0030.F207.6A8C** (SW-3). PC2 (10.0.0.10) hangs off SW-2 and PC3 (10.0.0.20) hangs off SW-3.

The port labels in the image show the following links. SW-1 Fa0/1 goes to SW-3 Fa0/1, and SW-1 Fa0/2 goes to SW-2 Fa0/1. SW-2 Fa0/2 goes to SW-3 Fa0/2, which is the redundant link. SW-2 Fa0/3 goes to PC2, and SW-3 Fa0/3 goes to PC3. An orange marker sits at the SW-2 end of the SW-2 to SW-3 link. In Packet Tracer this marks a port that STP has blocked, so SW-2 Fa0/2 is the blocked port.

The image shows the result of STP. The commands that produced the 4097 priority are not in the image or the notes, so they are covered in the Configuration section.

### Cabling

| From | Port | To | Port | Cable type |
|---|---|---|---|---|
| SW-1 | Fa0/1 | SW-3 | Fa0/1 | Crossover copper (switch to switch) |
| SW-1 | Fa0/2 | SW-2 | Fa0/1 | Crossover copper (switch to switch) |
| SW-2 | Fa0/2 | SW-3 | Fa0/2 | Crossover copper (switch to switch), redundant link |
| SW-2 | Fa0/3 | PC2 | Fa0 | Straight-through copper (switch to PC) |
| SW-3 | Fa0/3 | PC3 | Fa0 | Straight-through copper (switch to PC) |

The cable types follow the standard rule. The dashed lines in the image join switches to each other, and the solid lines join switches to PCs. Packet Tracer's auto-MDIX can hide the difference, so no screenshot confirms the cable types.

### Addressing Table

| Device | Interface | IP Address | Subnet Mask | Prefix | Default Gateway |
|---|---|---|---|---|---|
| PC2 | Fa0 | 10.0.0.10 | not recorded | /24 (assumed) | not recorded |
| PC3 | Fa0 | 10.0.0.20 | not recorded | /24 (assumed) | not recorded |

The switches have no IP addresses in the lab material. STP runs at Layer 2, so no switch IP is needed.

### Subnet Analysis

| Network | Mask | Network Address | First Usable | Last Usable | Broadcast | Usable Hosts | Private (RFC 1918)? |
|---|---|---|---|---|---|---|---|
| 10.0.0.0/24 | 255.255.255.0 (assumed) | 10.0.0.0 | 10.0.0.1 | 10.0.0.254 | 10.0.0.255 | 254 | Yes (10.0.0.0/8) |

The image gives the two host addresses but not the mask. Both addresses are in 10.0.0.0 to 10.0.0.255, so /24 is the most likely mask and is used here. Confirm it in the PC IP settings. Both PCs are in the same subnet and the same VLAN, so no gateway is needed for PC2 to reach PC3.

## 📚 Concepts Behind This Lab

**Why STP is needed.** Ethernet switches flood broadcast and unknown unicast frames out every port except the one they arrived on. In a loop, a broadcast is forwarded around the loop again and again, and the copies multiply. This is a broadcast storm. Because switches learn MAC addresses from frames, a looped frame also makes the MAC table change constantly. STP stops this by building a loop-free tree of forwarding ports across the switched network.

**Root bridge election.** Every switch sends BPDUs (Bridge Protocol Data Units). The switch with the lowest bridge ID becomes the root. The bridge ID is the priority followed by the MAC address. The default priority is 32768, and with the VLAN ID added (the extended system ID), the value is 32769 for VLAN 1. A lower priority always wins. If priorities tie, the lowest MAC address wins. This lab is a clear example. SW-1 has a lower priority (4097), so it is the root, and the MAC addresses are never compared.

**Port roles.** Each non-root switch chooses one **root port**, the port with the lowest cost path to the root. The root bridge has only **designated ports**. Each network segment has one designated port, which is the port that offers the lowest cost path to the root on that segment. Any port that is neither root nor designated becomes a **blocked port** and does not forward user traffic.

**Path cost.** The cost of a link depends on its speed. The notes list the values used on Cisco switches:

| Link speed | Cost |
|---|---|
| 10 Mbps Ethernet | 100 |
| 100 Mbps Fast Ethernet | 19 |
| 1 Gbps Gigabit Ethernet | 4 |
| 10 Gbps | 2 |

In this topology, every link is Fast Ethernet, so each direct link to the root costs 19. The path SW-2 to SW-1 costs 19, while the path SW-2 to SW-3 to SW-1 costs 19 + 19 = 38. The direct path wins.

**How the blocked port is chosen.** SW-2 and SW-3 each have a direct path to the root with cost 19, so their root-path costs tie. The tie is broken by bridge ID, which comes from priority and then MAC address. Both switches have priority 32769, so the MAC addresses decide. SW-3's MAC (0030.F207.6A8C) is lower than SW-2's (00E0.B08D.B7CA), so SW-3 wins that segment. SW-3 Fa0/2 becomes the designated port and forwards. SW-2 Fa0/2 is the non-designated port and is blocked. This matches the orange marker in the image.

**Port states (IEEE 802.1D).** A port moves through these states before it forwards traffic.

| State | Duration | What happens |
|---|---|---|
| Blocking | 20 seconds (max age) | Receives BPDUs but does not forward data or learn MAC addresses. |
| Listening | 15 seconds (forward delay) | Takes part in the election, sends and receives BPDUs, does not forward data. |
| Learning | 15 seconds (forward delay) | Builds the MAC table, does not forward data. |
| Forwarding | Indefinite | Forwards data and learns MAC addresses. |
| Disabled | Indefinite | Administratively shut down, takes no part in STP. |

The notes give 50 seconds as the total convergence time for classic STP. That total is 20 + 15 + 15 = 50 seconds. This slow convergence is the reason RSTP was developed.

**RSTP (802.1w).** Rapid STP uses three port states, discarding, learning and forwarding, and the discarding state covers both blocking and listening. RSTP adds the **alternate** and **backup** roles, which are ready to take over immediately. Convergence takes a few seconds rather than 50. In this lab, SW-2 Fa0/2 is the alternate port for SW-2. RSTP is enabled with `spanning-tree mode rapid-pvst`. Rapid PVST+ runs one spanning tree per VLAN, which is the Cisco default.

**PortFast.** An access port that connects to a PC should not run through the listening and learning states, because a PC is not a switch and cannot create a loop. PortFast moves the port directly to forwarding, so the PC gets a network connection as soon as it comes up. PortFast should only be used on ports connected to end devices.

**BPDU Guard.** BPDU Guard protects PortFast ports. If a BPDU arrives on a PortFast port, which means a switch was connected to a port meant for a PC, the port is shut down (err-disabled). This stops an attacker or a mistake from inserting a switch that could change the topology.

## 🧭 Lab Steps

1. Connect the switches in a triangle and the PCs to their access ports, as shown in the topology.
2. Assign IP addresses 10.0.0.10 (PC2) and 10.0.0.20 (PC3), from the image.
3. Make SW-1 the root bridge using `spanning-tree vlan 1 root primary` or a lower priority. The image shows the result, priority 4097.
4. Enable RSTP on each switch with `spanning-tree mode rapid-pvst`.
5. Enable PortFast and BPDU Guard on the PC-facing ports, Fa0/3 on SW-2 and SW-3. This step is a recommendation, see the Configuration notes.
6. Verify the root bridge, the port roles and the blocked port with `show spanning-tree`, and test PC2 to PC3 with a ping.

## ⚙️ Configuration

### Step 1: Make SW-1 the root bridge

No CLI screenshot was provided for this step. The result (SW-1 priority 4097) is shown in the topology image. The commands below come from the lab notes.

```
enable
configure terminal
spanning-tree vlan 1 root primary
end
write
```

| Command | Mode | What it does |
|---|---|---|
| `enable` | User EXEC | Enters privileged EXEC mode. |
| `configure terminal` | Privileged EXEC | Enters global configuration mode. |
| `spanning-tree vlan 1 root primary` | Global config | Lowers the bridge priority of this switch for VLAN 1 so that it becomes the root. |
| `end` | Global config | Returns to privileged EXEC mode. |
| `write` | Privileged EXEC | Saves the configuration. |

**Notes:** `root primary` sets a priority that is low enough to win the election, using a calculated value. The notes also give the manual form `spanning-tree vlan 1 priority 8192`. The image shows priority 4097, which is 4096 plus the VLAN ID of 1. That value is not the 8192 example in the notes, so the exact command used on SW-1 is not confirmed. Check the running configuration on SW-1 to see which command produced 4097. The root bridge should always be a core switch, and the other switches should keep the default priority.

### Step 2: Enable RSTP

No screenshot was provided for this step. The command below comes from the lab notes.

```
enable
configure terminal
spanning-tree mode rapid-pvst
end
write
```

| Command | Mode | What it does |
|---|---|---|
| `spanning-tree mode rapid-pvst` | Global config | Switches the spanning tree from classic PVST to Rapid PVST+ (802.1w). |

**Notes:** Set the mode on every switch in the loop, so that they all use the same rules. The mode is global, not per interface.

### Step 3: Enable PortFast and BPDU Guard on PC ports

No screenshot was provided for this step. The notes show PortFast on Fa0/1 as an example. Fa0/1 is a trunk to SW-1 in this topology, so PortFast is applied here to the PC-facing ports, Fa0/3, as a recommendation.

```
enable
configure terminal
interface FastEthernet0/3
 spanning-tree portfast
 exit
spanning-tree portfast bpduguard default
end
write
```

| Command | Mode | What it does |
|---|---|---|
| `interface FastEthernet0/3` | Global config | Selects the port that connects to the PC. |
| `spanning-tree portfast` | Interface config | Moves the port directly to forwarding for end devices. |
| `spanning-tree portfast bpduguard default` | Global config | Enables BPDU Guard on all PortFast-enabled ports. The notes give this as "enable globally". |

**Notes:** The global `bpduguard default` command applies to every PortFast port on the switch. Apply it on SW-2 and SW-3 so that a switch plugged into a PC port is shut down rather than forming a loop.

## ✅ Verification & Results

| Check | Command / Action | What success looks like | Result |
|---|---|---|---|
| Root bridge identity | `show spanning-tree` on SW-1 | "This bridge is the root", priority 4097 | Shown in topology image (priority and MAC) |
| Root on SW-2 and SW-3 | `show spanning-tree` on SW-2 and SW-3 | Root ID 4097, MAC 00E0.F9C0.3DAB | Expected (not captured) |
| Root port on SW-2 | `show spanning-tree` on SW-2 | Fa0/1 is Root, forwarding, cost 19 | Expected (not captured) |
| Root port on SW-3 | `show spanning-tree` on SW-3 | Fa0/1 is Root, forwarding, cost 19 | Expected (not captured) |
| Blocked link | `show spanning-tree` on SW-2 | Fa0/2 is Altn (alternate) and blocking | Shown in topology image (orange marker) |
| Designated port on the redundant link | `show spanning-tree` on SW-3 | Fa0/2 is Desg and forwarding | Expected (not captured) |
| RSTP mode | `show spanning-tree summary` | "rapid-pvst" listed as the mode (additional command, not in the notes) | Expected (not captured) |
| PortFast and BPDU Guard | `show spanning-tree interface fa0/3 detail` | "PortFast" and "bpdu guard" listed | Expected (not captured) |
| End-to-end path | `ping 10.0.0.20` from PC2 | Replies from PC3 (the first ping may time out while ARP resolves) | Expected (not captured) |

**Expected output (not captured)**, `show spanning-tree` on SW-2, showing where SW-1 is the root and which port is blocked. Use it to check what to look for, not as a record of this run:

```
VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    4097
             Address     00E0.F9C0.3DAB
             Cost        19
             Port        1 (FastEthernet0/1)

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00E0.B08D.B7CA

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Root FWD 19        128.1    P2p
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/3            Desg FWD 19        128.3    P2p Edge
```

The Root ID priority and address should match SW-1, and the cost to the root should be 19. Fa0/2 is blocked, so SW-2 sends no user traffic toward SW-3. The `Edge` label on Fa0/3 appears only if PortFast is enabled on that port.

## 🛠️ Troubleshooting Notes

**Problems the author hit:** None recorded. The notes and the image show the result with no errors.

**General STP faults (not observed in this lab):**

| Symptom | Likely cause | How to check | Fix |
|---|---|---|---|
| A switch other than SW-1 becomes root | SW-1 priority not lowered, or a switch with a lower priority was added | `show spanning-tree root` on each switch | Set the root with `root primary` and keep other switches at default |
| Broadcast storm, CPU high, MAC table changing | Loop with STP disabled on a port, or a port in the wrong mode | `show spanning-tree` on each switch and check for missing blocked ports | Enable STP and confirm the blocked port |
| PC port takes 30 to 50 seconds to get a link | Classic STP on an access port | `show spanning-tree interface fa0/3` | Enable PortFast on the PC port |
| PC port shows err-disabled | BPDU Guard triggered by a switch on a PortFast port | `show interfaces status err-disabled` | Remove the switch, then `shutdown` and `no shutdown` the port |
| Both links forwarding and traffic loops | STP disabled on one switch, or mismatched mode | Compare `show spanning-tree` output on all three switches | Use the same mode on every switch |
| Ping PC2 to PC3 fails after a link is removed | Blocked port not yet moved to forwarding | Watch `show spanning-tree` while the link is down | Wait for convergence, or use RSTP for faster recovery |

## 💼 Skills Demonstrated

- Built a three-switch triangle and explained why it creates a Layer 2 loop.
- Verified the root bridge election and matched the bridge IDs shown in the topology to the rules for priority and MAC address.
- Identified the root port on each non-root switch and the blocked port on the redundant link.
- Calculated path costs using the Fast Ethernet value of 19 and explained why the direct path wins.
- Explained the classic STP states and their timers, and why RSTP converges faster.
- Planned PortFast and BPDU Guard for the end-device ports to protect the network.

**CCNA 200-301 mapping:** 2.0 Network Access. Objective 2.5 (PVST+ spanning tree, configure and verify) and objective 2.6 (Rapid PVST+, configure and verify).

## 📂 Files in This Lab

```
STP/
├── README.md
├── STP___RSTP.JPG       # topology diagram with STP results
└── STP_Notes.txt        # original STP and RSTP notes
```

## 🚀 How to Open This Lab

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco NetAcad account).
2. Build the topology from the **Topology** section, with the three switches in a triangle and the PCs on their access ports.
3. On SW-1, open the **CLI** tab and enter the commands from Step 1. On all three switches, enter the commands from Steps 2 and 3.
4. On SW-2, run `show spanning-tree`. Fa0/1 should show Root and Fa0/2 should show Altn and blocking.
5. On PC2, open the **Command Prompt** and run `ping 10.0.0.20`.

## 🔭 Possible Improvements

- Turn off the link between SW-1 and SW-3 and watch SW-3 move its port to root through SW-2. This shows how STP recovers from a failure.
- Repeat the lab with classic 802.1D (`spanning-tree mode pvst`) and time the convergence with a continuous ping, then compare with RSTP.
- Add a fourth switch to see how a new switch with a lower priority would change the root.

## 👤 Author

Anish Raj — Aspiring Network Engineer, building CCNA / networking fundamentals skills.

---
*This lab is part of my ongoing CCNA / Network Administrator study roadmap.*
