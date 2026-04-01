# Day 10 — MQTT Broker Attacks: Injection & Fuzzing

## Setup
- Attacker: Kali Linux (192.168.56.20)
- Target: Ubuntu VM running Mosquitto MQTT broker (192.168.56.10)
- Tools: mosquitto-clients, paho-mqtt, Python3

## Step 1 — Verified Mosquitto Running Without Auth
Confirmed Mosquitto was active with default config (no authentication).
`sudo systemctl status mosquitto` showed active (running).

## Step 2 & 3 — Subscribed to All Topics (Wildcard)
```bash
mosquitto_sub -h 192.168.56.10 -t '#' -v &
```
Subscriber ran in background, listening to all topics in real time.

## Step 4 & 5 — Command Injection Attack
Published a fake unlock command to the smart lock topic:
```bash
mosquitto_pub -h 192.168.56.10 -t 'home/lock/frontdoor' -m 'UNLOCK'
```
Subscriber immediately received:
```
home/lock/frontdoor UNLOCK
```
This demonstrates that without authentication, any attacker on the network
can publish commands to any IoT device topic — including smart locks, alarms,
and sensors.

## Step 6 — Installed paho-mqtt
```bash
pip3 install paho-mqtt --break-system-packages
```
Version 2.1.0 was already present on Kali.

## Step 7 & 8 — Topic Fuzzer Script
Created mqtt-fuzz.py using paho-mqtt library. The script:
- Connects to the broker
- Subscribes to all topics with wildcard #
- Publishes FUZZ_TEST to 20 different topic variations
- Listens for 2 minutes and logs all discovered topics

## Step 9 — Discovered Topics
All 20 fuzzed topics were discovered and received by the subscriber:
- home/lock/frontdoor, home/lock/backdoor
- home/alarm/status, home/alarm/trigger
- home/temperature/living, home/temperature/bedroom
- home/lights/kitchen, home/lights/bedroom
- home/camera/feed, home/camera/control
- device/sensor/1, device/sensor/2
- iot/gateway/status, iot/gateway/command
- admin/config, admin/reset
- auth/login, auth/token
- sensor/temp, sensor/humidity

## Step 10 & 11 — Retained Message Attack
Published a retained message to the alarm topic:
```bash
mosquitto_pub -h 192.168.56.10 -t 'home/alarm/status' -m 'DISABLED' -r
```
After disconnecting and reconnecting the subscriber, the message was
immediately delivered again without anyone republishing it.

### Why This Is Dangerous
The broker permanently stores retained messages and delivers them to every
new subscriber automatically. An attacker can publish DISABLED to an alarm
topic with -r flag, then completely disconnect. Every future monitoring
system, app, or script that subscribes to that topic will believe the alarm
is disabled indefinitely — even hours or days after the attacker is gone.
This is a persistent poisoning attack requiring zero ongoing presence.

## AI Task — Microsoft Copilot
**Prompt used:** "Write a Python script using paho-mqtt that connects to an
MQTT broker, subscribes to all topics, and then publishes test messages to
20 different topic variations to enumerate what topics exist. The broker IP
should be a command-line argument."

Copilot generated the base script which was reviewed line by line, tested,
and annotated with comments explaining each section. Key observations:
- The script correctly used client.loop_start() for background message handling
- on_message callback was used to capture and deduplicate discovered topics
- sleep() delays between publishes prevented broker flooding
- The script was saved as scripts/mqtt-fuzz.py after review and annotation

## Key Findings
- Unauthenticated MQTT broker accepts commands from any network client
- Wildcard subscription # exposes all device topics to any attacker
- Retained messages persist on the broker indefinitely after attacker disconnects
- Topic enumeration via fuzzing reveals entire smart home device structure

## Mitigations
- Enable authentication: set allow_anonymous false and password_file in mosquitto.conf
- Use TLS on port 8883 to encrypt all MQTT traffic
- Implement topic-level ACLs to restrict publish/subscribe permissions
- Monitor and audit retained messages regularly<img width="608" height="211" alt="Screenshot 2026-04-01 120825" src="https://github.com/user-attachments/assets/b69732b2-ded4-4c0a-b359-f16956ac7471" />
<img width="760" height="727" alt="Screenshot 2026-04-01 130540" src="https://github.com/user-attachments/assets/98c0451e-b1d3-4368-b534-4c8f66ef6a22" />
