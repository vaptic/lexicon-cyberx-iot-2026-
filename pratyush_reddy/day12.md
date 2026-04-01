# Day 12 — STRIDE Methodology + OWASP Threat Dragon
---
https://github.com/vaptic/lexicon-cyberx-iot-2026-/blob/0baa8992b9c9bcc7279babeb096f5fd7cd434eee/pratyush_reddy/findings/threat-model-smart-home.json
---

## 📘 Theory Summary

### What is Threat Modelling?
Threat modelling is the process of systematically identifying all possible threats to a system before they are exploited. It answers four questions:
1. What are we building?
2. What can go wrong?
3. What are we going to do about it?
4. Did we do a good enough job?

### STRIDE Framework
STRIDE is a threat categorisation framework developed by Microsoft. Each letter represents a threat category:

| Letter | Threat | IoT Example |
|--------|--------|-------------|
| **S** | Spoofing | Fake MQTT client pretending to be a real sensor |
| **T** | Tampering | Injecting false temperature readings into the broker |
| **R** | Repudiation | No logging on IoT devices — can't prove who did what |
| **I** | Information Disclosure | Plaintext MQTT traffic readable by anyone on the network |
| **D** | Denial of Service | Flooding the MQTT broker to make it unavailable |
| **E** | Elevation of Privilege | Using hardcoded root credentials to gain admin access |

### What is OWASP Threat Dragon?
A free, browser-based threat modelling tool at threatdragon.com. You draw Data Flow Diagrams (DFDs) showing how data moves through a system, then tag each flow and component with STRIDE threats.

---

## 🔧 Practical Lab

### Threat Dragon Diagram — Smart Home IoT System

The following components were added to the diagram:

| Component Type | Name |
|---------------|------|
| External Entity (rectangle) | Smartphone App |
| Process (circle) | MQTT Broker |
| Process (circle) | IoT Sensor Device |
| Data Store (parallel lines) | Cloud Database |
| External Entity (rectangle) | Attacker |

### Data Flows Drawn

| Flow | Label |
|------|-------|
| IoT Sensor Device → MQTT Broker | Temp/sensor data |
| MQTT Broker → Cloud Database | Data storage |
| Smartphone App → MQTT Broker | Subscribe/send commands |
| Attacker → Cloud Database | Attack flows |

### Diagram Screenshot
<img width="1079" height="340" alt="Screenshot 2026-04-01 154333" src="https://github.com/user-attachments/assets/0b1a3ac7-d1f5-4618-b872-5dd10e5d6771" />



---

## 🔍 STRIDE Threat Analysis

### Flow: IoT Sensor Device → MQTT Broker
- 🔴 **Information Disclosure** — sensor data transmitted as plaintext, readable by any network listener
- 🔴 **Tampering** — attacker can inject false sensor readings (as demonstrated in Day 10)
- 🔴 **Spoofing** — a fake sensor can impersonate a real device with no authentication

### Flow: MQTT Broker → Cloud Database
- 🔴 **Tampering** — injected false data gets permanently stored in the database
- 🔴 **Information Disclosure** — unencrypted data stored without protection
- 🔴 **Denial of Service** — broker flooded with messages, no real data reaches cloud

### Flow: Smartphone App → MQTT Broker
- 🔴 **Spoofing** — attacker impersonates the smartphone app to send commands
- 🔴 **Tampering** — commands intercepted and modified via MitM (as demonstrated in Day 11)
- 🔴 **Elevation of Privilege** — attacker sends admin-level commands without authorisation

### Flow: Attacker → MQTT Broker / Cloud
- 🔴 **Denial of Service** — flood the broker to make it unavailable
- 🔴 **Spoofing** — publish as a legitimate device
- 🔴 **Information Disclosure** — subscribe to `#` wildcard and read all topics

### Component: MQTT Broker
- 🔴 **Repudiation** — no logging by default, impossible to prove who published what
- 🔴 **Denial of Service** — broker can be overwhelmed with connections
- 🔴 **Elevation of Privilege** — no authentication means anyone can publish or subscribe

### Component: IoT Sensor Device
- 🔴 **Tampering** — physical access to device allows hardware manipulation
- 🔴 **Spoofing** — device identity can be cloned or faked

### Component: Cloud Database
- 🔴 **Information Disclosure** — data stored without encryption
- 🔴 **Tampering** — corrupted data stored from injected MQTT messages

---

## 🤖 AI Tool Task — ARP Spoofing Analogy

**Prompt given to ChatGPT:**
> "Explain how ARP spoofing works as an analogy — compare it to something from everyday life that a student would understand."

**Response:**

### 🏠 The College Hostel Delivery Analogy

Every student has:
- A **name** → like an IP address
- A **room number** → like a MAC address

**Normal situation:** A delivery person asks "Which room is Rahul in?" — Rahul replies "I'm in Room 101" and the package arrives correctly.

**ARP Spoofing situation:** A malicious student intercepts and says "Hey, I'm Rahul — I'm in Room 202." The delivery person trusts the reply and sends the package to the wrong room.

The attacker in Room 202 can then:
- Open and read the package 📦
- Modify the contents ✏️
- Forward it to the real Rahul so no one notices 😈

| Real Life | Network Concept |
|-----------|----------------|
| Student name | IP address |
| Room number | MAC address |
| Asking for room | ARP request |
| Fake response | ARP spoofing |
| Fake student | Attacker |

**Why it's dangerous:** Devices trust ARP replies blindly with no verification — making the attacker a silent man-in-the-middle who can spy on data, modify messages, and steal passwords.

> **One-line summary:** "ARP spoofing is like someone pretending to be your friend's room and receiving all your deliveries before passing them on."

---

## 🎯 Outcomes

- ✅ STRIDE framework understood and applied to Smart Home IoT system
- ✅ Threat Dragon diagram created with all 5 components and 4 data flows
- ✅ All data flows tagged with relevant STRIDE threats
- ✅ ARP spoofing analogy documented from AI tool task
- ✅ Findings committed to GitHub

---

## 🛠 Tools Used

| Tool | Purpose |
|------|---------|
| OWASP Threat Dragon (threatdragon.com) | DFD diagram and STRIDE tagging |
| ChatGPT | ARP spoofing analogy explanation |

