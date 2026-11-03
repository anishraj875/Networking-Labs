[← Back to all labs](../README.md)

# Configuring SSH on a Cisco Router (R1) (Cisco Packet Tracer)

This lab sets up a Cisco IOS router so that it accepts encrypted remote administration over SSH version 2 instead of clear-text Telnet. A PC on the same LAN then logs in to the router over SSH and reads its configuration.

## 📋 At a Glance

| | |
|---|---|
| **Lab number** | [N] |
| **Level** | Beginner, CCNA |
| **Tools** | Cisco Packet Tracer, Cisco IOS CLI, PC Command Prompt |
| **Devices** | 1 router (R1, model not recorded), 1 switch (model not recorded), 1 PC (PC1) |
| **Protocols / features** | SSH version 2, RSA keys, local user authentication, VTY lines, TCP 22 |
| **Network(s)** | 192.168.1.0/24 |
| **CCNA domains** | Security Fundamentals, Network Fundamentals (IP addressing) |

## 📌 Project Objective

Configure R1 so that a PC on its LAN can open an encrypted remote session to the router using SSH version 2, log in with a local username and password, and run privileged commands. The success test is a working SSH login from PC1 to 192.168.1.1, which is shown in the screenshots below.

## 🌍 Real-World Scenario

Network devices are usually managed remotely, often from a different room or site. Telnet sends the username, password and every command in clear text, so anyone capturing traffic on the path can read them. SSH encrypts the whole session, which is why it is the standard for managing routers and switches in production networks. Enterprise networks typically go further and use AAA servers (TACACS+ or RADIUS) and management VLANs, but the core steps (hostname, domain name, RSA key, SSH version 2, VTY lines) are the same on every Cisco device.

## 🖧 Topology

![Network topology showing R1, a switch and PC1 with IP addresses](Images/SSH.JPG)

The topology has three devices in a straight line. R1 connects to the switch on its Gig0/0 interface, and the switch connects to PC1 on Fa0/1. The image is labeled "Configuring SSH on R1" and lists the addresses for R1 (192.168.1.1) and PC1 (192.168.1.10), both with mask 255.255.255.0. The switch has no IP address in this design because it only forwards frames at Layer 2.

### Cabling

| From | Port | To | Port | Cable type |
|---|---|---|---|---|
| R1 (router) | Gig0/0 | Switch | Gig0/1 | Straight-through (standard for router-to-switch; cable type not visible in the image) |
| Switch | Fa0/1 | PC1 | Fa0 | Straight-through (standard for switch-to-PC; cable type not visible in the image) |

### Addressing Table

| Device | Interface | IP Address | Subnet Mask | Prefix | Default Gateway |
|---|---|---|---|---|---|
| R1 | Gig0/0 | 192.168.1.1 | 255.255.255.0 | /24 | Not applicable (router interface) |
| PC1 | Fa0 | 192.168.1.10 | 255.255.255.0 | /24 | Not listed in the topology |
| Switch | (VLAN 1 SVI) | Not configured | Not configured | n/a | Not applicable |

### Subnet Analysis

| Network | Mask | Network Address | First Usable | Last Usable | Broadcast | Usable Hosts | Private (RFC 1918)? |
|---|---|---|---|---|---|---|---|
| 192.168.1.0/24 | 255.255.255.0 | 192.168.1.0 | 192.168.1.1 | 192.168.1.254 | 192.168.1.255 | 254 | Yes (192.168.0.0/16) |

Both 192.168.1.1 (R1) and 192.168.1.10 (PC1) fall inside 192.168.1.0/24. The router takes the first usable address, .1, which is a common convention because it makes the gateway easy to find. Because PC1 and R1 are on the same subnet, PC1 reaches the router directly and does not need a default gateway for this test.

## 📚 Concepts Behind This Lab

**Telnet versus SSH.** Telnet (TCP port 23) sends everything in clear text, including passwords. SSH (TCP port 22) encrypts the session. Any Cisco device that supports remote management should use SSH version 2, because SSH version 1 has known weaknesses.

**RSA keys and why the domain name is needed.** SSH needs a public/private key pair on the router. IOS names the key after the hostname and domain, which is why `hostname R1` and `ip domain-name lab.local` must be set before `crypto key generate rsa`. The result is the key name R1.lab.local seen in the screenshot. Generating the key also turns SSH on, which is why the log line "SSH 1.99 has been enabled" appears. The key modulus controls key strength. IOS suggests 512 bits by default, but SSH version 2 requires at least 768 bits, and 2048 is the recommended value used in this lab.

**Local authentication with privilege 15.** `username <name> privilege 15 secret <password>` creates a user in the router's local database and stores the password as a hash. `login local` on the VTY lines tells the router to check that database. Privilege level 15 gives full administrative rights, so the user lands directly in privileged EXEC (the `R1#` prompt) after login.

**VTY lines.** Remote sessions arrive on virtual terminal lines, numbered 0 to 4 here (five sessions). `transport input ssh` allows only SSH on those lines. Without it, Telnet is the default on many IOS versions, so this command is what actually closes the clear-text door.

| Method | Port | Encrypted? | Use in this lab |
|---|---|---|---|
| Telnet | TCP 23 | No | Not configured |
| SSH | TCP 22 | Yes | Configured on VTY 0 to 4 |

## 🧭 Lab Steps

1. Set the hostname, disable DNS lookups, and configure and enable the router's Gig0/0 interface with 192.168.1.1/24.
2. Set the domain name and generate a 2048-bit RSA key pair.
3. Create a local user with privilege 15 and a secret password.
4. Enable SSH version 2 and configure the VTY lines to accept SSH and check the local user database.
5. From PC1, open an SSH session to 192.168.1.1, log in, and display the running configuration.

## ⚙️ Configuration

### Step 1: Hostname and Gig0/0 interface

![Router CLI showing hostname R1, interface Gig0/0 configured with 192.168.1.1 and the line protocol coming up](Images/Int_IP_and_Host.JPG)

```
enable
configure terminal
hostname R1
no ip domain-lookup
interface gigabitEthernet 0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
end
```

| Command | Mode | What it does |
|---|---|---|
| `enable` | User EXEC (`Router>`) | Moves to privileged EXEC (`Router#`). |
| `configure terminal` | Privileged EXEC | Enters global configuration mode (`Router(config)#`). |
| `hostname R1` | Global config | Renames the router. The prompt changes to `R1(config)#`. |
| `no ip domain-lookup` | Global config | Stops the router from trying to resolve mistyped commands as DNS names, which avoids long delays. |
| `interface gigabitEthernet 0/0` | Global config | Enters interface configuration mode (`R1(config-if)#`). |
| `ip address 192.168.1.1 255.255.255.0` | Interface config | Assigns the router's LAN address and mask. |
| `no shutdown` | Interface config | Brings the interface up. Router interfaces start administratively down, so this step is required. |
| `end` | Any config mode | Returns to privileged EXEC. |

**Notes:** The screenshot shows the author typed the shortened forms `int`, `ip add`, `no sh` and `no ip dom`. IOS accepts these abbreviations as long as they are unambiguous. The log messages `%LINK-5-CHANGED` and `%LINEPROTO-5-UPDOWN` confirm that the interface moved to up. The `Router#` prompt is privileged EXEC, so the `enable` and `configure terminal` commands were run in that order. The screenshot shows the author's commands at the exact point where the interface came up.

### Step 2: Domain name and RSA key

![Router CLI showing ip domain-name, crypto key generate rsa with 2048 bits, and SSH enabled](Images/SSH_config.JPG)

```
ip domain-name lab.local
crypto key generate rsa
! IOS prompt: How many bits in the modulus [512]: 2048
```

| Command | Mode | What it does |
|---|---|---|
| `ip domain-name lab.local` | Global config | Sets the domain name. IOS combines it with the hostname to name the RSA key (R1.lab.local). |
| `crypto key generate rsa` | Global config | Creates the RSA key pair. IOS asks for the modulus size. |
| `2048` (answer to the prompt) | Key generation prompt | Sets the key size to 2048 bits instead of the 512-bit default. |

**Notes:** The key must be generated after the hostname and domain name are set. The screenshot shows "% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]", which means the key was created. The key is stored in the router's configuration, not as a file you can copy off the device. Generating keys can take a few minutes on a real router.

### Step 3: Local user and SSH version 2

Continuing in the same session from Step 2:

```
username anishraj privilege 15 secret <password>
ip ssh version 2
line vty 0 4
transport input ssh
login local
exit
```

| Command | Mode | What it does |
|---|---|---|
| `username anishraj privilege 15 secret <password>` | Global config | Creates a local user with full privileges. The `secret` keyword stores the password as a hash. |
| `ip ssh version 2` | Global config | Allows only SSH version 2 connections. |
| `line vty 0 4` | Global config | Enters configuration for the five virtual terminal lines used by remote sessions. |
| `transport input ssh` | Line config (`R1(config-line)#`) | Accepts only SSH on these lines, which blocks Telnet. |
| `login local` | Line config | Checks logins against the local username database. |
| `exit` | Line config | Returns to global configuration mode. |

**Notes:** The screenshot shows the log line `%SSH-5-ENABLED: SSH 1.99 has been enabled`, which appears after the username command. The `1.99` value means SSH is enabled and both version 1 and version 2 are allowed until `ip ssh version 2` restricts it to version 2. The password is masked in this README (`<password>`) because it is a lab credential that is published in a public repository. The screenshot shows it, so see the note in the closing message. If `login local` is missing, IOS falls back to asking for a line password, and because no line password exists, login fails. The `transport input ssh` command is the most important line for security because it closes Telnet on the same lines.

### Step 4: Test the SSH login from PC1

![PC1 Command Prompt showing ssh -l anishraj 192.168.1.1, the password prompt, and show running-config output on R1](Images/Succesfully_Login_Via_SSH.JPG)

```
C:\>ssh -l anishraj 192.168.1.1
Password:
R1#en
R1#show running-config
```

| Command | Mode / Where | What it does |
|---|---|---|
| `ssh -l anishraj 192.168.1.1` | PC1 Command Prompt | Opens an SSH session to 192.168.1.1 as user anishraj. `-l` sets the login name. |
| `en` | Router privileged EXEC | Confirms privileged mode (`enable` abbreviated). |
| `show running-config` | Router privileged EXEC | Displays the active configuration in RAM. The screenshot shows "Current configuration : 782 bytes" and `hostname R1`. |

**Notes:** The command prompt on PC1 is not an IOS command. It is the Windows-style prompt Packet Tracer provides on PCs. The `Password:` prompt confirms the router accepted the SSH connection and is asking for the local password. After login, the router prompt `R1#` shows the session is in privileged EXEC because the user has privilege 15.

## ✅ Verification & Results

| Check | Command / Action | What success looks like | Result |
|---|---|---|---|
| Interface up | Link and line protocol messages after `no shutdown` | Gig0/0 changes to up, and line protocol changes to up | Shown in screenshot |
| SSH enabled | Log message after the username command | `%SSH-5-ENABLED` message | Shown in screenshot |
| Remote login | `ssh -l anishraj 192.168.1.1` from PC1 | Password prompt, then a router session | Shown in screenshot |
| Configuration review | `show running-config` in the SSH session | Hostname R1 and the config is visible | Shown in screenshot (partial, first lines only) |
| SSH status | `show ip ssh` on R1 | Version 2.0 enabled | Expected (not captured) |
| Reachability | `ping 192.168.1.1` from PC1 | Successful Echo Replies | Expected (not captured) |

**Captured output (from the screenshots):**

```
%LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]
*Mar 1 0:6:0.221: %SSH-5-ENABLED: SSH 1.99 has been enabled
```

**Expected output (not captured, for checking your own lab):**

```
R1# show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 3
```

## 🛠️ Troubleshooting Notes

No problems were recorded for this lab. The table below lists the most likely failures for an SSH lab and how to check each one, working from Layer 1 up to the application layer.

| Symptom | Likely cause | How to check | Fix |
|---|---|---|---|
| PC cannot reach the router at all | Interface down, wrong IP or mask, cable or port problem | `show ip interface brief` on R1; `ping 192.168.1.1` from PC1 | `no shutdown` on Gig0/0; correct the address; check cable and switch port |
| `Connection refused` or SSH never starts | RSA key not generated, or no `ip domain-name` set | `show ip ssh`; `show running-config` for `crypto` and `ip domain-name` | Set `ip domain-name`, then `crypto key generate rsa` |
| Password prompt never appears | `transport input ssh` missing or wrong VTY range | `show running-config` section for `line vty` | Add `transport input ssh` under `line vty 0 4` |
| Login rejected after password | Wrong username or password, or `login local` missing | Check the username spelling; `show running-config` for `login local` | Retype the credentials; add `login local` to the VTY lines |
| Session logs in but lands in user mode | Privilege level below 15 | `show running-config` for `username ... privilege` | Recreate the user with `privilege 15` |

## 💼 Skills Demonstrated

- Configured SSH version 2 on a Cisco IOS router to replace clear-text Telnet for remote administration.
- Generated a 2048-bit RSA key pair and set the domain name it depends on.
- Created a local privileged user and applied local authentication to VTY lines.
- Restricted VTY access to SSH only with `transport input ssh`.
- Verified the configuration by logging in from a PC over SSH and checking the running configuration.
- Documented each command with its mode and purpose so the lab can be repeated.

**CCNA 200-301 mapping:** 5.0 Security Fundamentals (device access control, SSH, local password and privilege configuration) and 1.0 Network Fundamentals (IPv4 addressing and subnetting for 192.168.1.0/24).

## 📂 Files in This Lab

```
[Lab folder name]/
├── README.md
├── Images/
│   ├── SSH.JPG                          # Topology diagram
│   ├── Int_IP_and_Host.JPG              # R1 hostname and Gig0/0 configuration
│   ├── SSH_config.JPG                   # Domain name, RSA key and SSH configuration
│   └── Succesfully_Login_Via_SSH.JPG    # Successful SSH login from PC1
└── [Packet Tracer file].pkt             # Lab file (if saved)
```

## 🚀 How to Open This Lab

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco NetAcad account).
2. Open the `.pkt` file for this lab, if included, or rebuild the topology from the diagram above.
3. Click R1, open the **CLI** tab, and enter the commands in the Configuration section. Then click PC1, open the **Desktop** tab, choose **Command Prompt**, and run `ssh -l <username> 192.168.1.1`.

## 🔭 Possible Improvements

- Add a separate management VLAN and restrict SSH to the admin PC's address with an access control list on the VTY lines.
- Replace the local user database with AAA on a RADIUS or TACACS+ server in Packet Tracer.
- Add `exec-timeout` and `login block-for` so idle sessions end and repeated failed logins are slowed down.
- Add a second PC on another subnet, with a router-on-a-stick or a second router, to test SSH across a routed path.

## 👤 Author

[Your Name] — Aspiring Network Engineer, building CCNA / networking fundamentals skills.

---
*This lab is part of my ongoing CCNA / Network Administrator study roadmap.*
