# Basic Switch Configuration & Telnet Remote Access Lab

> **Tool:** Cisco Packet Tracer  |  **Level:** CCNA Beginner  |  **Topic:** Switch basics, management IP, secret password, Telnet

---

## 1. Overview

This lab shows how to take a brand-new Cisco switch and make it **manageable over the network**.
You will give the switch a management IP address, protect privileged mode with an enable secret,
and enable **Telnet** so you can log in remotely from a PC instead of using the console cable.

## 2. Objectives

1. Configure a switch (hostname)
2. Assign an IP address to the switch (management interface)
3. Enable a secret password for privileged EXEC mode
4. Configure Telnet (VTY lines) and test remote login from a PC

## 3. Topology

![Network topology](images/01-topology.jpg)

| Device | Model | Interface | IP Address | Subnet Mask | Connected To |
|--------|-------|-----------|------------|-------------|--------------|
| SW1 | Cisco 2960-24TT | VLAN 1 (SVI) | 20.0.0.1 | 255.0.0.0 | - |
| PC1 | PC-PT | Fa0 | 20.0.0.2 | 255.0.0.0 | SW1 Fa0/1 |
| PC2 | PC-PT | Fa0 | 20.0.0.3 | 255.0.0.0 | SW1 Fa0/2 |

Cables: copper straight-through (PC Fa0 to switch Fa0/1 and Fa0/2).

## 4. Prerequisites

- Cisco Packet Tracer installed
- Basic idea of CLI modes: User EXEC (`>`), Privileged EXEC (`#`), Global config (`(config)#`)
- Open `Basic_Switch_Telnet.pkt` to see the finished lab, or build it yourself with the steps below

## 5. Configuration Steps

### Step 1: Configure the switch (hostname)

```
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
```

### Step 2: Assign an IP address to the switch

A Layer 2 switch has no IP on its physical ports. The IP goes on the **VLAN 1 SVI**
(Switch Virtual Interface), which is the management interface.

```
SW1(config)# interface vlan 1
SW1(config-if)# ip address 20.0.0.1 255.0.0.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

When the interface comes up you will see `Line protocol on Interface Vlan1, changed state to up`.

### Step 3: Enable the secret password

```
SW1(config)# enable secret <your-secret-password>
```

- `enable secret` is stored as an **MD5 hash** (type 5), so it is safer than `enable password`.

### Step 4: Configure Telnet (VTY lines)

```
SW1(config)# line vty 0 4
SW1(config-line)# password <your-vty-password>
SW1(config-line)# login
SW1(config-line)# exit
```

- `line vty 0 4` selects the 5 virtual terminal lines (5 simultaneous Telnet sessions).
- `password` sets the password asked at the Telnet prompt.
- `login` makes the switch actually ask for that password.

**Screenshot of the switch CLI (IP address + Telnet configuration):**

![Switch configuration](images/02-switch-configuration.jpg)

### Step 5: Configure the PCs

On each PC go to **Desktop > IP Configuration** and set:

| PC | IP Address | Subnet Mask |
|----|------------|-------------|
| PC1 | 20.0.0.2 | 255.0.0.0 |
| PC2 | 20.0.0.3 | 255.0.0.0 |

### Step 6: Save the configuration

```
SW1# copy running-config startup-config
```

## 6. Verification

### 6.1 Test Telnet from a PC

On a PC open **Desktop > Command Prompt** and run:

```
C:\> telnet 20.0.0.1
```

Expected flow:

1. `Trying 20.0.0.1 ...Open` means the TCP connection works
2. `Password:` is the **VTY password** (Step 4)
3. `SW1>` means you are in User EXEC mode
4. Type `en`, then the **enable secret** (Step 3), and you get `SW1#`

![Telnet verification](images/03-telnet-verification.jpg)

### 6.2 Check the running configuration

```
SW1# show running-config
```

Look for:

- `hostname SW1`
- `enable secret 5 $1$...` (hashed, so it is not readable)
- `interface Vlan1` with `ip address 20.0.0.1 255.0.0.0`
- `line vty 0 4` with a password

### 6.3 Other useful commands

| Command | Purpose |
|---------|---------|
| `show ip interface brief` | Check that Vlan1 is up/up with the right IP |
| `show running-config` | View the active configuration |
| `show users` | See who is connected via Telnet |
| `ping 20.0.0.1` | Test connectivity from a PC |

## 7. How It Works

```
PC (20.0.0.2) --- telnet 20.0.0.1 ---> SW1 Vlan1 SVI --> VTY line --> VTY password --> enable --> enable secret --> SW1#
```

The PC and the switch are in the same network (20.0.0.0/8), so no router is needed.
The PC reaches the switch's VLAN 1 IP on TCP port 23, and the VTY line handles the session.

## 8. Troubleshooting

| Problem | Likely Cause | Fix |
|---------|--------------|-----|
| `Request timed out` / Telnet fails | Vlan1 is shut down or wrong IP | `no shutdown` on Vlan1, check `show ip interface brief` |
| Telnet connects, then `Password required, but none set` | No VTY password configured | Set `password` under `line vty 0 4` |
| Cannot reach `SW1#` | Enable secret not set or mistyped | Re-enter `enable secret` |
| Ping fails | PC IP or mask wrong, or cable on wrong port | Recheck PC IP Configuration and link lights |

## 9. Security Note

- **Telnet sends everything, including passwords, in clear text.** It is fine for a lab, but real networks should use **SSH**.
- The VTY password shows in plain text in `show running-config` unless you run `service password-encryption`.
- Never publish real passwords in a README. Use placeholders as shown above.

## 10. Key Takeaways

- A Layer 2 switch is managed through an **SVI** (`interface vlan 1`), not its physical ports.
- `enable secret` protects privileged mode and is hashed.
- `line vty 0 4` plus `password` and `login` enables remote Telnet access.
- Telnet works inside the same network without a router.

## 11. Files in This Repository

```
Basic-Switch-Telnet-Lab/
├── README.md
├── Basic_Switch_Telnet.pkt        # Packet Tracer lab file
└── images/
    ├── 01-topology.jpg
    ├── 02-switch-configuration.jpg
    └── 03-telnet-verification.jpg
```

## 12. Author

**Anish** ([@anishraj875](https://github.com/anishraj875)) - CCNA learner, building one lab at a time.
