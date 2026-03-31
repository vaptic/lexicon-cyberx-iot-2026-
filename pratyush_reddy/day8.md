# Day 8 — IoT Traffic Capture & Deep Packet Analysis

**Phase:** Phase 3 — Network Threat Analysis  
**Date:** 2026-03-31  
**Ratio:** 30% Theory / 70% Practical

---

## Lab Environment

| Role | OS | IP Address |
|---|---|---|
| Attacker (Sniffer) | Kali Linux | 192.168.56.102 |
| IoT Target (Broker) | Ubuntu Server 24.04 LTS | 192.168.56.101 |
| Network Interface | Host-Only Adapter | eth1 |
| MQTT Broker | Mosquitto 2.0.18 | Port 1883 |

---

## Theory Summary

### Why IoT Traffic Capture Matters
A packet capture reveals everything about a device's behaviour:
- What protocols the device uses
- Whether data is encrypted or transmitted in plaintext
- What credentials are transmitted
- Device fingerprints (model/brand/firmware)
- Timing patterns of communication

### Wireshark: Capture Filters vs Display Filters
- **Capture filter** — limits what packets are captured before they are recorded (faster, applied at capture time)
- **Display filter** — limits what you see in already-captured data (applied after capture)

Key display filters used in this lab:
- `mqtt` — shows only MQTT traffic
- `tcp.port == 1883` — filters by MQTT default port
- `ip.addr == 192.168.56.101` — filters by device IP

### Protocol Dissection
Wireshark understands 3,000+ protocols. Each MQTT packet breaks down into layers:
```
Ethernet Frame
  └── IP Packet
        └── TCP Segment
              └── MQTT Application Layer
                    ├── Topic
                    ├── Message (payload)
                    ├── QoS Level
                    └── Authentication (if any)
```

---

## Step-by-Step Lab

### Step 1 — Start Mosquitto on Ubuntu
```bash
sudo systemctl start mosquitto
sudo systemctl status mosquitto
```
✅ Mosquitto running on Ubuntu 24.04 LTS at 192.168.56.101:1883

### Step 2 — Configure Mosquitto for External Connections
The default Mosquitto config only listens on localhost. Added to `/etc/mosquitto/mosquitto.conf`:
```
listener 1883
allow_anonymous true
```
```bash
sudo systemctl restart mosquitto
```

### Step 3 — Start Wireshark on Kali
- Opened Wireshark on Kali Linux
- Selected interface: **eth1** (host-only adapter on 192.168.56.x subnet)
- Started live capture

### Step 4 — Publish Simulated IoT Sensor Messages
Ran from Kali using `mosquitto_pub` with `-h 192.168.56.101` to force traffic over the network:
```bash
mosquitto_pub -h 192.168.56.101 -t 'home/bedroom/temp' -m '24.5'
mosquitto_pub -h 192.168.56.101 -t 'home/lock/status' -m 'UNLOCKED'
mosquitto_pub -h 192.168.56.101 -t 'home/camera/auth' -m 'user=admin&pass=admin123'
```

### Step 5 — Apply MQTT Display Filter
Applied filter `mqtt` in Wireshark. The following packets were captured:

| Packet | Source | Destination | Info |
|---|---|---|---|
| Connect Command | 192.168.56.102 | 192.168.56.101 | Client connecting to broker |
| Connect Ack | 192.168.56.101 | 192.168.56.102 | Broker accepting connection |
| Publish Message | 192.168.56.102 | 192.168.56.101 | `home/lock/status` → UNLOCKED |
| Publish Message | 192.168.56.102 | 192.168.56.101 | `home/camera/auth` → **user=admin&pass=admin123** |
| Disconnect Req | 192.168.56.101 | 192.168.56.102 | Session ended |

---

## Key Finding — Plaintext Credentials Captured

### Screenshot Evidence
The packet for topic `home/camera/auth` was selected and expanded in the MQ Telemetry Transport Protocol layer, revealing:

```
MQ Telemetry Transport Protocol
  ├── Header Flags: 0x30, Message Type: Publish Message
  ├── Msg Len: 42
  ├── Topic Length: 16
  ├── Topic: home/camera/auth
  └── Message: user=admin&pass=admin123        ← PASSWORD IN PLAINTEXT
```

**The password `admin123` was transmitted in completely unencrypted plaintext over the network.**  
Any device on the same network running Wireshark could capture this credential with zero effort.

---

## Follow TCP Stream

Right-clicked the credential packet → Follow → TCP Stream. The full conversation was reconstructed:

```
....MQTT...<..
...
0*..home/camera/authuser=admin&pass=admin123..
```

This shows the **entire MQTT session as a readable conversation** — the protocol header, the topic name, and the credential payload are all visible as plain ASCII text. There is no encryption layer whatsoever.

**3 client packets, 1 server packet, 2 turns** captured in this stream.

---

## Protocol Hierarchy Analysis

From **Statistics → Protocol Hierarchy**:

| Protocol | Percent Packets | Packets | Percent Bytes |
|---|---|---|---|
| Frame | 100.0% | 13 | 100.0% |
| Ethernet | 100.0% | 13 | 19.1% |
| Internet Protocol Version 4 | 100.0% | 13 | 27.3% |
| Transmission Control Protocol | 100.0% | 13 | 46.6% |
| **MQ Telemetry Transport (MQTT)** | **30.8%** | **4** | **6.7%** |

**MQTT accounted for 30.8% of all packets** in this capture session — a significant proportion, meaning an attacker monitoring the network would immediately notice MQTT as a primary protocol in use.

---

## Security Issues Identified

| # | Issue | Severity | Detail |
|---|---|---|---|
| 1 | Plaintext credentials | 🔴 Critical | `user=admin&pass=admin123` visible in packet payload |
| 2 | No TLS/SSL encryption | 🔴 Critical | Entire MQTT session readable in Wireshark |
| 3 | Anonymous access enabled | 🔴 Critical | No authentication required to connect to broker |
| 4 | Sensitive topics exposed | 🟠 High | Lock status, camera auth, temperature all in plaintext |
| 5 | Default credentials used | 🟠 High | admin/admin123 is a commonly targeted default combination |

---

## AI Traffic Analysis

**Tool used:** Google Gemini  
**Prompt:** *"I captured this IoT network traffic. Identify all security issues you can find in this traffic pattern and explain each one."*

**AI Analysis Summary:**
- Identified unencrypted MQTT as the primary risk — all messages transmitted without TLS
- Flagged the `home/camera/auth` topic as especially dangerous — camera authentication credentials should never be transmitted in plaintext
- Noted that `home/lock/status: UNLOCKED` being broadcast publicly creates a physical security risk
- Recommended implementing MQTT over TLS (port 8883), enabling username/password authentication in Mosquitto, and using certificate-based client authentication for sensitive IoT devices

---

## Expected Outcomes — Completed

- ✅ Wireshark capture showing MQTT credentials in plaintext
- ✅ TCP stream reconstruction completed
- ✅ Protocol hierarchy analysis completed (`mqtt` = 30.8% of packets)
- ✅ Traffic security report written

---

## Files


<img width="1366" height="720" alt="Screenshot 2026-03-31 121547" src="https://github.com/user-attachments/assets/89b48c31-92aa-493a-bdaa-0f243a5dc665" />
<img width="1366" height="720" alt="Screenshot 2026-03-31 121811" src="https://github.com/user-attachments/assets/efaa43cf-725c-4d9e-9885-1e91696d5639" />
<img width="1366" height="720" alt="Screenshot 2026-03-31 121825" src="https://github.com/user-attachments/assets/04abb116-73a0-41a7-b025-9c0b9073aa50" />
