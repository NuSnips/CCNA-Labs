# Lab 01: Foundational Switch Security & Management

## 📌 Project Overview

**Target Certification:** Cisco CCNA 200-301  
**Status:** Completed & Verified  
**Difficulty:** ⭐☆☆☆☆  
**Environment:** Cisco Packet Tracer 8.x

### Objective

To transform a "factory-default" switch into a production-ready device by implementing industry-standard security hardening, remote management (SSH), and administrative documentation.

---

## 🗺️ Network Topology

| Device | Role          | Management IP   | VLAN | Interface      |
| :----- | :------------ | :-------------- | :--- | :------------- |
| SW1    | Access Switch | 192.168.1.11/24 | 1    | F0/24 (To SW2) |
| SW2    | Access Switch | 192.168.1.12/24 | 1    | F0/24 (To SW1) |
| PC1    | Host          | 192.168.1.1/24  | 1    | SW1 F0/1       |
| PC2    | Host          | 192.168.1.2/24  | 1    | SW1 F0/2       |
| PC3    | Host          | 192.168.1.3/24  | 1    | SW2 F0/1       |
| PC4    | Host          | 192.168.1.4/24  | 1    | SW2 F0/2       |

---

## 🛠️ Implementation Checklist

### 1. Global Device Hardening

Prevent CLI "freezes" and secure the privileged EXEC mode.

```ios
SW1(config)# hostname SW1
SW1(config)# no ip domain-lookup
SW1(config)# enable secret class456
SW1(config)# service password-encryption

```

- **Why `no ip domain-lookup`?** Stops the switch from attempting DNS resolution on mistyped commands, preventing a 60-second CLI lockout.
- **Why `enable secret`?** Uses MD5 hashing (Type 5) which is far more secure than the plaintext `enable password`.

### 2. Management IP Configuration (SVI)

Assigning the switch its "identity" on the network.

```
SW1(config)# interface vlan 1
SW1(config-if)# ip address 192.168.1.11 255.255.255.0
SW1(config-if)# no shutdown
SW1(config)# ip default-gateway 192.168.1.10
```

### 3. Secure Remote Access (SSH v2)

Replacing insecure Telnet with encrypted SSH.

```ios
SW1(config)# ip domain-name my-lab.local
SW1(config)# crypto key generate rsa
  > How many bits in the modulus [512]: 2048
SW1(config)# ip ssh version 2
SW1(config)# username admin privilege 15 secret admin789

```

### 4. Line-Level Security

Applying timeouts and preventing message interruptions.

```ios
SW1(config)# line con 0
SW1(config-line)# logging synchronous
SW1(config-line)# exec-timeout 5 0
SW1(config-line)# login local

SW1(config)# line vty 0 15
SW1(config-line)# transport input ssh
SW1(config-line)# login local

```

- **`logging synchronous`**: Reprints your current command line if a system log message appears.
- **`transport input ssh`**: Specifically disables Telnet to prevent plain-text credential sniffing.

---

## ✅ Verification Results

### SSH Connectivity Test (From Admin-PC)

Successful remote login confirms RSA key generation and VTY line configuration are correct.

### Verification Commands

| Command                 | Purpose              | Expected Output                  |
| ----------------------- | -------------------- | -------------------------------- |
| `show ip ssh`           | Verify Version       | SSH Enabled - version 2.0        |
| `show ip int brief`     | Check Management IP  | Vlan1 ... 192.168.1.11 ... up/up |
| `show run section line` | Verify Line Security | line con 0...logging synchronous |

---

## 🐛 Troubleshooting

**Issue:** SSH connection refused from Admin-PC.

**Discovery:** Running `show ip ssh` returned "%SSH has not been enabled".

**Root Cause:** RSA keys were not generated because a `domain-name` had not been set.

**Resolution:** Configured `ip domain-name lab.local`, generated 2048-bit keys, and SSH became active immediately.

**Lesson Learned:** SSH is a multi-step dependency chain (Management IP -> Hostname -> Domain Name -> RSA Keys).

---

## 📚 Key Skills Mastered

- **Device Hardening:** Implementing MD5 secrets and Type-7 service encryption.
- **Cryptographic Operations:** Deploying RSA key pairs for transport security.
- **In-Band Management:** Configuring SVI (Switch Virtual Interface) for remote IP access.
- **Documentation:** Maintaining professional-grade configuration logs.

---
