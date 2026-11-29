[← Back to all labs](../README.md)

# VTP (VLAN Trunking Protocol) Lab: Server, Client and Transparent Modes (Cisco Packet Tracer)

This lab configures three switches in VTP domain `ccna` and shows how the three VTP modes behave. A VLAN created on the server switch is advertised over trunk links to the client, while the transparent switch forwards the advertisements without learning from them.

## 📋 At a Glance

| | |
|---|---|
| **Lab number** | [N] |
| **Level** | Intermediate, CCNA |
| **Tools** | Cisco Packet Tracer, Cisco IOS CLI |
| **Devices** | 3 Layer 2 switches (SW-1 Server, SW-2 Client, SW-3 Transparent). Model not recorded in the screenshots. |
| **Protocols / features** | VTP version 2, 802.1Q trunking, VLAN database |
| **Network(s)** | None. This is a Layer 2 lab with no IP addressing. |
| **VLANs** | VLAN 10 (HR), VLAN 20 (MR), created on the server |
| **CCNA domains** | 2.0 Network Access |

## 📌 Project Objective

Configure VTP on three switches so that they share one VTP domain and one password, with each switch set to a different VTP mode. Success means that the server's VLAN changes appear on the client and the transparent switch, which is shown with `show vtp status` and `show vlan brief`. The lab notes state three rules: every link between switches must be a trunk, the domain name must match on all devices, and the password must match.

## 🌍 Real-World Scenario

In a campus or enterprise network, dozens of switches often carry the same set of VLANs. Without VTP, an engineer would have to type each VLAN onto every switch, and a missed switch would drop traffic for that VLAN. VTP lets an administrator create or rename a VLAN once on a server switch, and every client in the domain receives the change automatically. Network teams use this to keep VLAN databases consistent, although many also turn VTP off or use VTP transparent on some switches because a badly configured VTP change can remove VLANs across a whole network.

## 🖧 Topology

![VTP topology with SW-1 (Server), SW-2 (Client) and SW-3 (Transparent) connected in a line over trunk links](VTP.JPG)

The three switches are connected in a line. The topology labels SW-1 as **Server**, SW-2 as **Client** and SW-3 as **Transparent**, and marks the two inter-switch links as **TrunkPort**. The topology shows SW-1 Fa0/1 linked to SW-2 Fa0/1, and SW-2 Fa0/2 linked to SW-3 Fa0/1. The configuration screenshots use different port numbers on SW-2 and SW-3. This is covered in the questions at the end of this README.

No IP addresses are used, so the Addressing Table and Subnet Analysis sections from the template do not apply to this lab.

### Cabling

| From | Port | To | Port | Cable type |
|---|---|---|---|---|
| SW-1 (Server) | Fa0/1 | SW-2 (Client) | Fa0/1 | Copper, crossover (switch to switch) |
| SW-2 (Client) | Fa0/2 | SW-3 (Transparent) | Fa0/1 | Copper, crossover (switch to switch) |

The cable type is not visible in the topology image. The table shows the standard rule for two switches (like devices use crossover copper). Packet Tracer's auto-MDIX can make the difference invisible, so the cable type is not confirmed by a screenshot.

## 📚 Concepts Behind This Lab

**Why VTP exists.** VTP is a Cisco protocol that carries VLAN information (VLAN IDs and names) between switches over trunk links. It keeps VLAN databases in sync, so a VLAN created on one switch appears on the others in the same domain. VTP only carries VLAN information. It does not carry port assignments, so each switch still needs its access ports assigned to the right VLAN.

**VTP domain, password and version.** Switches must share the same VTP domain name to exchange advertisements. A password adds an MD5 check so that switches ignore advertisements from an unauthorised source. Both the domain name and the password are case-sensitive, so `CCNA` and `ccna` are different values. VTP version 2 supports the same features as version 1, adds consistency checks, and supports Token Ring. A client switch cannot change its version, as the lab screenshots show.

**VTP modes.** Each switch runs in one of three modes, and the mode decides whether it can change the VLAN database.

| Mode | Create, modify, delete VLANs | Learn VLANs from advertisements | Forward advertisements from other switches | Typical use |
|---|---|---|---|---|
| Server | Yes | Yes | Yes | Central place to manage VLANs. Default mode on most switches. |
| Client | No | Yes | Yes | Switches that only receive VLAN changes. |
| Transparent | Yes (local only) | No | Yes (version 2 only) | Switches that keep their own VLANs and pass VTP messages through. |

**Configuration revision number.** Each VLAN change increases the revision number, and switches accept an advertisement only when its revision is higher than their own. This is the reason VTP can cause problems: a switch that was used elsewhere with a higher revision number can overwrite the VLAN database of an entire domain when it joins. A common practice is to reset the revision number before adding a switch.

**Why the links must be trunks.** VTP advertisements travel in frames tagged for the trunk, so they only cross links that carry multiple VLANs. An access port blocks them, which is why the lab rule says every link must be a trunk.

## 🧭 Lab Steps

1. Connect the three switches in a line, as shown in the topology: SW-1 Fa0/1 to SW-2 Fa0/1, and SW-2 Fa0/2 to SW-3 Fa0/1.
2. Configure SW-1 as the VTP server, with its link to SW-2 as a trunk.
3. Configure SW-2 as a VTP client, with its link to SW-1 and SW-3 as trunks.
4. Configure SW-3 as a VTP transparent switch, with its link to SW-2 as a trunk.
5. Create VLAN 10 (HR) and VLAN 20 (MR) on SW-1, which is the server.
6. Verify that the VLANs and VTP settings appear on SW-2 and SW-3.

## ⚙️ Configuration

### Step 1: Configure SW-1 as the VTP server

![SW-1 CLI output showing trunk, VTP domain, server mode, version 2 and password](SW-1_Config.JPG)

*Caption: SW-1 CLI session. The trunk is configured on Fa0/1, the device is set to VTP server mode with domain `ccna`, and the configuration is saved with `write memory`.*

```
enable
configure terminal
hostname SW-1
interface FastEthernet0/1
 switchport mode trunk
 exit
vtp domain ccna
vtp mode server
vtp version 2
vtp password ccna@8835
end
write memory
```

| Command | Mode | What it does |
|---|---|---|
| `enable` | User EXEC | Enters privileged EXEC mode. |
| `configure terminal` | Privileged EXEC | Enters global configuration mode. |
| `hostname SW-1` | Global config | Sets the device name shown in the prompt. The screenshot shows it typed as `host SW-1`, which IOS accepted. |
| `interface FastEthernet0/1` | Global config | Selects the port that connects to SW-2. |
| `switchport mode trunk` | Interface config | Makes the port a trunk, so it carries VLAN traffic and VTP advertisements. |
| `vtp domain ccna` | Global config | Sets the VTP domain name. The screenshot confirms "Changing VTP domain name from NULL to ccna". |
| `vtp mode server` | Global config | Sets the VTP mode. The screenshot shows "Device mode already VTP SERVER", because server is the default. |
| `vtp version 2` | Global config | Sets VTP version 2. |
| `vtp password ccna@8835` | Global config | Sets the password used to authenticate VTP advertisements. |
| `write memory` | Privileged EXEC | Saves the running configuration to startup configuration. The screenshot shows `[OK]`. |

**Notes:** On most switches, server is the default VTP mode, so the explicit command confirms the setting rather than changing it. The VTP domain must be set before the password, because the password is stored for the domain. The lab notes spell the domain and password in capital letters (`CCNA`, `CCNA@8835`). The CLI session used lowercase (`ccna`, `ccna@8835`) on all three switches, so the lab works with the values in the screenshots. The password above is for a practice lab only.

### Step 2: Configure SW-2 as the VTP client

![SW-2 CLI output showing trunk, VTP domain already set, client mode, and the rejected version command](SW-2_Configs.JPG)

*Caption: SW-2 CLI session. The domain is already `ccna`, the device enters VTP client mode, and the version command is rejected because client mode cannot change the version.*

```
enable
configure terminal
hostname SW-2
interface FastEthernet0/2
 switchport mode trunk
 exit
vtp domain ccna
vtp mode client
vtp version 2
vtp password ccna@8835
end
write memory
```

| Command | Mode | What it does |
|---|---|---|
| `hostname SW-2` | Global config | Names the switch SW-2. |
| `interface FastEthernet0/2` | Global config | Selects the port that connects to SW-3. |
| `switchport mode trunk` | Interface config | Makes the port a trunk so VTP messages can reach SW-3. |
| `vtp domain ccna` | Global config | Sets the domain. The screenshot shows "Domain name already set to ccna." because the switch had already learned the domain. |
| `vtp mode client` | Global config | Sets the client mode. The screenshot shows "Setting device to VTP CLIENT mode." |
| `vtp version 2` | Global config | Rejected. The screenshot shows "Cannot modify version in VTP client mode." |
| `vtp password ccna@8835` | Global config | Sets the password. The screenshot shows "Setting device VLAN database password to ccna@8835." |
| `write memory` | Privileged EXEC | Saves the configuration. The screenshot shows `[OK]`. |

**Notes:** A client cannot change the VTP version, so the command is rejected. The client takes the version from the server, so the line in the notes can be left out without affecting the lab. The screenshot shows the password accepted. The lab notes also contain the typo `fa0/2e`. The CLI session used `FastEthernet0/2`, and that is the port used here.

### Step 3: Configure SW-3 as the VTP transparent switch

![SW-3 CLI output showing trunk, VTP domain, transparent mode, version 2 and password](SW-3_Config.JPG)

*Caption: SW-3 CLI session. The port is set to trunk, the device enters VTP transparent mode, and the configuration is saved.*

```
enable
configure terminal
hostname SW-3
interface FastEthernet0/2
 switchport mode trunk
 exit
vtp domain ccna
vtp mode transparent
vtp version 2
vtp password ccna@8835
end
write memory
```

| Command | Mode | What it does |
|---|---|---|
| `hostname SW-3` | Global config | Names the switch SW-3. |
| `interface FastEthernet0/2` | Global config | Selects the port used for the trunk. The topology shows SW-3 on Fa0/1, which is covered in the questions. |
| `switchport mode trunk` | Interface config | Makes the port a trunk. |
| `vtp domain ccna` | Global config | Sets the domain. The screenshot confirms "Changing VTP domain name from NULL to ccna." |
| `vtp mode transparent` | Global config | Sets the transparent mode. The screenshot shows "Setting device to VTP TRANSPARENT mode." |
| `vtp version 2` | Global config | Sets version 2. |
| `vtp password ccna@8835` | Global config | Sets the password. |
| `write memory` | Privileged EXEC | Saves the configuration. The screenshot shows `[OK]`. |

**Notes:** On the first attempt, the mode keyword was typed as `transpa`, then entered again in full as `transparent`. The screenshot shows the full keyword was accepted. A transparent switch keeps its own VLAN database, so it forwards VTP advertisements but does not learn VLANs from them.

### Step 4: Create VLANs on the server (testing)

No screenshot was provided for this step. The commands below come from the lab notes.

```
enable
configure terminal
vlan 10
 name HR
vlan 20
 name MR
end
write memory
```

| Command | Mode | What it does |
|---|---|---|
| `vlan 10` | Global config | Creates VLAN 10 and enters VLAN configuration. |
| `name HR` | VLAN config | Names VLAN 10 `HR`. |
| `vlan 20` | Global config | Creates VLAN 20. |
| `name MR` | VLAN config | Names VLAN 20 `MR`. |

**Notes:** VLANs are created on the server. The client and the transparent switch should show the same VLANs after the advertisement is received. This is the test that shows VTP working. The VLAN names are copied from the notes as written.

## ✅ Verification & Results

| Check | Command / Action | What success looks like | Result |
|---|---|---|---|
| SW-1 is server | `vtp mode server` on SW-1 | "Device mode already VTP SERVER" | Shown in screenshot |
| SW-2 is client | `vtp mode client` on SW-2 | "Setting device to VTP CLIENT mode" | Shown in screenshot |
| SW-3 is transparent | `vtp mode transparent` on SW-3 | "Setting device to VTP TRANSPARENT mode" | Shown in screenshot |
| Domain matches on SW-2 | `vtp domain ccna` on SW-2 | "Domain name already set to ccna" | Shown in screenshot |
| Configs saved | `write memory` on each switch | `[OK]` after "Building configuration" | Shown in screenshots |
| Trunk on SW-1 | `show interfaces trunk` | Fa0/1 listed as trunking | Expected (not captured) |
| VTP status on each switch | `show vtp status` | Matching domain, version 2, correct mode | Expected (not captured) |
| VLANs reach the client | `show vlan brief` on SW-2 | VLAN 10 HR and VLAN 20 MR listed | Expected (not captured) |
| VLANs reach the transparent switch | `show vlan brief` on SW-3 | VLAN 10 HR and VLAN 20 MR listed | Expected (not captured) |

**Expected output (not captured)**, the `show vtp status` output on SW-2 after the VLANs are created. Use it to check what to look for, not as a record of this run:

```
VTP Version capable             : 1 to 3
VTP version running             : 2
VTP Domain Name                 : ccna
VTP Pruning Mode                : Disabled
VTP Operating Mode              : Client
VTP Password                    : ccna@8835 (lab only)
Number of existing VLANs        : 7
Configuration Revision          : (non-zero, increases after each change)
```

The VLAN count is 7 because the five default VLANs (1 and 1002 to 1005) are present, plus VLANs 10 and 20. The revision number increases on each change, and it matches on the server and client when they are in sync.

## 🛠️ Troubleshooting Notes

**Problems the author hit (shown in the screenshots):**

- The version command was rejected on SW-2 with "Cannot modify version in VTP client mode." This is expected behaviour for a client, not a fault.
- The transparent mode keyword was first typed as `transpa`. It was entered again in full, and the screenshot shows the full command was accepted.

**General VTP faults (not observed in this lab):**

| Symptom | Likely cause | How to check | Fix |
|---|---|---|---|
| Client does not receive VLANs | VTP domain name does not match | `show vtp status` on each switch | Set the same domain name (case-sensitive) |
| Client does not receive VLANs | VTP password does not match | `show vtp password` and the advertisement messages | Set the same password on every switch |
| Client does not receive VLANs | Link is an access port, not a trunk | `show interfaces trunk` | Set `switchport mode trunk` on both ends |
| Client shows no VLANs after creating them | Change was made on the client or transparent switch | `show vtp status` for the mode | Make VLAN changes on the server only |
| VLANs disappear across the domain | A switch with a higher revision number joined | Compare `Configuration Revision` on each switch | Reset the revision number on the new switch before it joins |

## 💼 Skills Demonstrated

- Configured VTP server, client and transparent modes on three switches and explained what each mode can do.
- Set a matching VTP domain name and password across a multi-switch domain.
- Configured 802.1Q trunk links between switches so VTP advertisements reach every device.
- Read IOS responses to confirm each change, and identified a rejected command (client version) as expected behaviour.
- Saved configurations on every device with `write memory` after each stage.
- Planned a VLAN creation test on the server to confirm propagation to the other switches.

**CCNA 200-301 mapping:** 2.0 Network Access. Objective 2.1 (configure and verify VLANs spanning multiple switches) and objective 2.2 (configure and verify interswitch connectivity using trunk ports). VTP itself is a Cisco protocol covered as background and is not named in the objective list.

## 📂 Files in This Lab

```
VTP/
├── README.md
├── Readme.txt          # original lab notes and command list
├── VTP.JPG             # topology diagram
├── SW-1_Config.JPG     # SW-1 CLI screenshot (server)
├── SW-2_Configs.JPG    # SW-2 CLI screenshot (client)
└── SW-3_Config.JPG     # SW-3 CLI screenshot (transparent)
```

## 🚀 How to Open This Lab

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco NetAcad account).
2. Build the topology from the **Topology** section above, with the three switches connected in a line.
3. On each switch, open the **CLI** tab and enter the commands in the **Configuration** section, in order.
4. On SW-1, create VLAN 10 and VLAN 20 as shown in Step 4.
5. On SW-2 and SW-3, run `show vtp status` and `show vlan brief` to confirm the VLANs arrived.

## 🔭 Possible Improvements

- Turn on VTP pruning so that VLAN traffic is only flooded to switches that need it. This is useful when a domain has more switches.
- Repeat the lab with VTP version 3, which adds the primary server feature and protects the VLAN database from accidental overwrites.
- Add a fourth switch as a client to test that a new switch learns the full VLAN database without any manual VLAN entry.

## 👤 Author

Anish Raj — Aspiring Network Engineer, building CCNA / networking fundamentals skills.

---
*This lab is part of my ongoing CCNA / Network Administrator study roadmap.*
