# Day 8 - IoT Traffic Capture & Deep Packet Analysis

## Lab Setup
- Kali Linux VM (Attacker) - 192.168.56.101
- Ubuntu IoT Target VM - 192.168.56.10
- Mosquitto MQTT broker running on Ubuntu port 1883

## What I Did
- Captured MQTT traffic using Wireshark on eth1
- Published simulated IoT sensor messages from Kali to Ubuntu
- Filtered packets using `mqtt` display filter
- Found plaintext credentials in packet 8 (Publish Message)

## Findings
- MQTT sends all data in plaintext with zero encryption
- Credentials visible in packet: `user=admin&pass=admin123`
- Topic: `home/camera/auth`
- Any attacker on the same network can capture these credentials instantly

## TCP Stream
- Followed TCP stream on the MQTT publish packet
- Full conversation visible including CONNECT, PUBLISH and DISCONNECT

## Protocol Hierarchy
- Captured traffic breakdown shows MQTT running over TCP/IP
- No encryption layer present anywhere in the stack

## AI Traffic Analysis
Gemini identified the following security issues in the captured traffic:
- **Plaintext credentials** - username and password transmitted with no encryption, visible to anyone on the network
- **No TLS/SSL** - MQTT running on port 1883 without encryption, should use port 8883 with TLS
- **Anonymous or weak authentication** - credentials like admin/admin123 indicate default or weak passwords
- **Sensitive topic names** - topics like `home/camera/auth` and `home/lock/status` reveal device structure to attackers
- **No access control** - any client can subscribe to any topic and intercept all messages<img width="947" height="305" alt="Screenshot 2026-03-31 132821" src="https://github.com/user-attachments/assets/cee7f8c2-bca9-4051-b473-69ab21aefc24" />
<img width="866" height="458" alt="Screenshot 2026-03-31 132542" src="https://github.com/user-attachments/assets/7b6a27ca-e6b2-4f4f-b368-f4ed3901b2f7" />
<img width="935" height="820" alt="Screenshot 2026-03-31 132449" src="https://github.com/user-attachments/assets/a7311f1b-0865-433e-b73a-02245650caea" />
<img width="932" height="767" alt="Screenshot 2026-03-31 132423" src="https://github.com/user-attachments/assets/b3d66efd-20cf-402d-83e3-46728a569284" />
