# IoT Communication Protocols & Security Research

## Day 3 — MQTT Protocol & Wireshark Packet Analysis

### What I Did
Installed and configured the Mosquitto MQTT broker on Kali Linux, simulated an IoT sensor
network using publish/subscribe messaging, and captured the traffic in Wireshark to analyse
the raw packets.

### Tools Used
- **Mosquitto** — open source MQTT broker and client tools
- **Wireshark** — network packet analyser

### Steps Completed
1. Installed Mosquitto broker and clients via apt
2. Started the broker in verbose mode on port 1883
3. Subscribed to all topics using the `#` wildcard
4. Published a test message to the `home/temperature` topic
5. Confirmed message delivery in the subscriber terminal
6. Captured live traffic in Wireshark on the `lo` loopback interface
7. Applied the `mqtt` display filter to isolate MQTT packets
8. Clicked into a PUBLISH packet and expanded the MQTT layer
9. Found `Topic: home/temperature` and the message payload in plaintext
10. Saved the capture as `day3-mqtt-capture.pcap`

### Key Finding
MQTT on port 1883 transmits data with zero encryption. The topic and message payload are
fully visible in Wireshark as plaintext — no decryption needed. In a real IoT deployment
this means anyone on the network could read sensor data, device states, or commands sent
to smart devices.

### Key Concepts Learned
- **Pub/Sub architecture** — publishers and subscribers never communicate directly,
  everything routes through the broker
- **Topics** — act as channels/addresses for messages (e.g. `home/temperature`)
- **Wildcard `#`** — subscribes to every topic at once
- **Port 1883** — default unencrypted MQTT port
- **Port 8883** — MQTT over TLS (encrypted, the secure alternative)
- **Loopback interface (`lo`)** — where local traffic between processes on the same
  machine flows through

---

# IoT Communication Protocols: MQTT, CoAP, Zigbee, BLE

These are communication protocols — rules devices use to talk to each other.

---

## 1. MQTT (Message Queuing Telemetry Transport)

Works like a message broker system. Devices communicate with each other through a middle man.

- **Publisher** – Who sends the data
- **Subscriber** – Who receives the data
- **Broker** – The middle man server (all messages go through here)

> **Example:** A temperature sensor says `Temp = 30°C` → sends to broker → broker sends
> to all subscribers

- Messages are organised like folders: `home/livingroom/temp`

### Why Hackers Like MQTT
- Often no authentication
- Broker may be publicly exposed
- Data is in plain text
- Can be sniffed with Wireshark (sees plaintext)

---

## 2. CoAP (Constrained Application Protocol)

- Used for **request-response communication**
- Instead of `GET /temp`, we do: `CoAP GET → temp`

### Basic Operations
| Method | Action |
|--------|--------|
| GET | Get data |
| POST | Send data |
| PUT | Update |
| DELETE | Remove |

> **Example:** Asking a smart bulb its status (on/off)

- CoAP is **faster but less reliable** compared to MQTT
- Runs on **UDP**; uses small packets

### Security Issues
- No encryption by default
- Easy to spoof (fake identity) requests
- Device may expose endpoints

---

## 3. BLE (Bluetooth Low Energy)

Short-range wireless communication between nearby devices using very low power.

> **Examples:** Smartwatches, earbuds, fitness bands, nearby pairing, wearables,
> smart locks, medical devices, tracking devices (like tags)

### Why Different from Regular Bluetooth?
- Uses very low power
- Slower than regular Bluetooth
- Used in **IoT sensors**

### How It Works
- **Peripheral (Server)** – The device that sends data (e.g. smartwatch)
- **Central (Client)** – The device that receives data (e.g. phone)

### Communication Flow
1. If not connected, sends a notification
2. Phone sees it → connects → exchanges data
3. That sending notification is a concept called **advertising**

### Structure
- **Service** = category (e.g. heart rate)
- **Characteristic** = actual data (e.g. 98bpm)

### Security Issues
1. No authentication — some devices allow anyone to connect
2. **Data sniffing** → BLE signal can be captured
3. **Replay attack** → attacker records signal & replays it; device thinks it's legit
4. **Device spoofing** → can create fake BLE device

---

## 4. Zigbee

Low-power wireless network where devices talk to each other directly, but forms a
**mesh network** to communicate.

- Mesh network = instead of connecting every device to one router, devices connect
  to each other

### Structure
1. **Coordinator** – Starts the network; controls everything
2. **Router** – Passes messages; extends range *(note: router ≠ WiFi router — it's a
   Zigbee device that passes messages)*
3. **End Device** – Sensor, bulb, etc.; doesn't pass messages

> **Path example:** Phone → Hub → Device A → Device B → Bulb

### Why Zigbee?
- Very low power
- Long range
- Reliable even if one node fails

### Security Issues
1. **Weak encryption** → attacker can read data
2. **Key sharing problems** → if network key leaks, whole network is compromised
3. **Device takeover** → attackers join network and control devices

   ---

# AI Task — MQTT vs HTTPS Security

## MQTT vs HTTPS for IoT Data Transmission

### Simple Analogy
- **MQTT** is like a walkie-talkie — tune in once, stay connected, lightweight and efficient
- **HTTPS** is like a phone call — dial every time, more secure by default but too heavy
  for tiny IoT devices

MQTT uses a persistent connection so a sensor can send thousands of messages with a single
handshake, making it ideal for low-power devices sending frequent small data. HTTPS opens
and closes a new connection every request, which is fine for occasional large transfers like
firmware updates but too heavy for a tiny temperature sensor firing every 5 seconds.

The catch with MQTT is that encryption is optional — you have to deliberately add TLS.
HTTPS has encryption basically built in by default. So MQTT is more efficient but needs
extra work to be secure, while HTTPS is secure out of the box but too resource-heavy for
most IoT devices.

> **Takeaway:** Use MQTT + TLS for sensors, HTTPS for firmware updates and API calls.

---

## The MQTT Wildcard Subscription Attack

The `#` wildcard subscribes to every topic on a broker at once. On a misconfigured broker
this becomes a serious attack vector.

### How It Works
1. Search Shodan for exposed brokers on port 1883
2. Connect anonymously — many brokers ship with `allow_anonymous true`
3. Subscribe to `#` and instantly receive every message on every topic
4. Read sensitive data like door lock states, sensor readings, device commands
5. Worse — publish to those topics and actually control the devices

> Over 47,000 MQTT brokers are publicly exposed on the internet right now, many with
> anonymous access enabled.

### Mitigations
- Never use `allow_anonymous true` in production
- Enforce ACLs so each device can only access its own topics
- Always use TLS on port 8883, never plain 1883
- Never expose your broker directly to the internet

### Connection to Today's Lab
The anonymous broker + `#` wildcard + Wireshark plaintext capture we did today is the
**exact same attack chain** — just on localhost instead of a real target.
