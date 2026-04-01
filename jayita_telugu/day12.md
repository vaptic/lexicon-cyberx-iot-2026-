# Day 12 — STRIDE Threat Modelling with OWASP Threat Dragon

## Setup
- Tool: OWASP Threat Dragon (threatdragon.com — browser based, no install)
- Methodology: STRIDE (Spoofing, Tampering, Repudiation, Information 
  Disclosure, Denial of Service, Elevation of Privilege)

## What is Threat Modelling
Threat modelling is the process of systematically identifying all possible
threats to a system before they are exploited. It answers four questions:
- What are we building?
- What can go wrong?
- What are we going to do about it?
- Did we do a good enough job?

## STRIDE Framework
| Category | Description | IoT Example |
|----------|-------------|-------------|
| Spoofing | Faking identity | Fake MQTT client impersonating a sensor |
| Tampering | Modifying data | Injecting false temperature readings |
| Repudiation | Denying actions | No logging on IoT devices |
| Information Disclosure | Leaking data | Plaintext MQTT credentials |
| Denial of Service | Flooding/crashing | Flooding the MQTT broker |
| Elevation of Privilege | Gaining more access | Using default creds for admin access |

## Step 1 — Created Model in Threat Dragon
- Visited threatdragon.com
- Created new model named: Smart Home IoT System
- Created new diagram named: Smart Home Data Flow
- Diagram type: Data Flow Diagram

## Step 2 — Components Added
| Component | Type |
|-----------|------|
| Smartphone App | External Entity |
| MQTT Broker | Process |
| IoT Sensor Device | Process |
| Cloud Database | Data Store |
| Attacker | External Entity |

## Step 3 — Data Flows Added
| Flow | Description |
|------|-------------|
| IoT Sensor → MQTT Broker | Temperature data (plaintext) |
| MQTT Broker → Cloud Database | Data storage |
| Smartphone → MQTT Broker | Subscribe / commands |
| Attacker → MQTT Broker | Injected commands (unauthenticated) |

## Step 4 — STRIDE Threats Tagged

### Sensor → MQTT Broker Flow
| Threat | STRIDE Category | Description |
|--------|----------------|-------------|
| Plaintext credential exposure | Information Disclosure | MQTT CONNECT packets transmit credentials in plaintext. Interceptable by any network observer. |
| Sensor data injection | Tampering | Attacker publishes false sensor readings to trigger incorrect device responses. |
| Fake sensor spoofing | Spoofing | Without client auth, any device can publish to the sensor topic impersonating a real sensor. |

### MQTT Broker Process
| Threat | STRIDE Category | Description |
|--------|----------------|-------------|
| Broker flood | Denial of Service | Attacker floods broker with thousands of publish requests exhausting resources. |
| Unauthenticated access | Elevation of Privilege | Default Mosquitto config requires no authentication giving any client full broker access. |

### Smartphone App
| Threat | STRIDE Category | Description |
|--------|----------------|-------------|
| Account takeover | Spoofing | Attacker obtains app credentials and impersonates the homeowner. |
| No audit trail | Repudiation | Commands sent via app are not logged so malicious actions cannot be traced. |

### Cloud Database
| Threat | STRIDE Category | Description |
|--------|----------------|-------------|
| Unauthorized data access | Information Disclosure | Historical sensor data exposed if DB has no access controls. |
| Data corruption | Tampering | Injected fake data stored permanently corrupting historical records. |

### MQTT Broker → Cloud DB Flow
| Threat | STRIDE Category | Description |
|--------|----------------|-------------|
| Data interception in transit | Information Disclosure | If cloud sync uses HTTP not HTTPS all data is readable in transit. |

### Attacker → MQTT Broker Flow
| Threat | STRIDE Category | Description |
|--------|----------------|-------------|
| Command injection | Tampering | Attacker publishes UNLOCK to lock topics as demonstrated in Day 10. |
| Retained message poisoning | Tampering | Attacker plants retained messages that persist after disconnection. |

## Step 5 — Mitigations

| Threat | Mitigation |
|--------|-----------|
| Information Disclosure (plaintext) | Enable TLS on MQTT — use port 8883. Configure Mosquitto with certfile, keyfile, cafile. |
| Spoofing (fake sensor) | Require client certificates (mutual TLS). Each device gets unique cert, broker rejects uncertified clients. |
| Tampering (data injection) | Implement topic-level ACLs — sensors can only publish to their own topic. |
| Denial of Service (broker flood) | Set max_connections in Mosquitto config. Use rate limiting and firewall rules. |
| Elevation of Privilege (unauth) | Set allow_anonymous false and password_file in mosquitto.conf. |
| Repudiation (no logging) | Enable Mosquitto logging. Forward logs to centralized SIEM for audit trail. |
| Retained message poisoning | Restrict retained message publishing via ACLs. Monitor retained message changes. |
| Cloud data interception | Use HTTPS/TLS for all cloud API calls. Never transmit IoT data over plain HTTP. |

## Step 6 — Exported Model
Model exported as: threat-models/smart-home-iot.json

## AI Task — Google Gemini STRIDE Analysis
**Prompt used:** "I have a smart home IoT system with a Smartphone App,
MQTT Broker, IoT Sensor Device, Cloud Database, and an Attacker as external
entities. For the MQTT channel between sensor and broker, what STRIDE threats
apply? Give me specific, realistic attack scenarios for each of the 6 STRIDE
categories."

**Gemini response summary and comparison:**

Gemini identified the following threats for the Sensor → MQTT Broker channel:

- Spoofing: A malicious device on the network publishes fake temperature
  readings to the sensor's topic pretending to be the legitimate sensor.
  Since MQTT has no device identity verification by default, the broker
  accepts it without question. This matched what was already in the model.

- Tampering: An attacker intercepts the MQTT connection using ARP spoofing
  (exactly as demonstrated in Day 11) and modifies sensor values mid-transit
  before they reach the broker — for example changing 25°C to 99°C to
  trigger the cooling system constantly. This was already covered.

- Repudiation: Gemini highlighted that most IoT sensors have no logging
  capability whatsoever. If a sensor publishes a malicious command there
  is no way to prove which device sent it or when. This was added to the
  model as it was initially missed.

- Information Disclosure: Gemini specifically called out that MQTT CONNECT
  packets expose clientID, username and password in plaintext — directly
  proven in Day 11 where admin/secret123 was captured with tcpdump.
  Already in model.

- Denial of Service: Gemini suggested a specific attack scenario where an
  attacker subscribes to all topics with wildcard # and simultaneously
  floods the broker with 10,000 publish messages per second using a script,
  causing legitimate sensors to lose connection. Added max_connections
  mitigation based on this.

- Elevation of Privilege: Gemini pointed out that gaining broker access
  via default credentials not only allows reading all topics but also
  allows modifying broker configuration remotely if the management port
  is exposed. This specific detail was added to the model.

**Additional threats added after Gemini review:**
- Repudiation threat on Sensor device (no logging capability)
- Specific EoP scenario around exposed broker management port

## Key Findings
- All 6 STRIDE categories demonstrated with real smart home IoT examples
- Every threat directly maps to attacks performed in Days 10 and 11
- Retained message attack from Day 10 = Tampering in STRIDE
- Credential interception from Day 11 = Information Disclosure in STRIDE
- Threat modelling connects all previous practical work into a structured framework

## Conclusion
STRIDE threat modelling transforms ad-hoc security testing into a systematic
and repeatable process. By mapping every component and data flow to specific
threat categories, security gaps become visible before exploitation rather
than after. For the Smart Home IoT system analysed, all 6 STRIDE categories
were present — confirming that default MQTT configurations are comprehensively
insecure and require TLS, authentication, ACLs and logging as minimum baseline
controls.[threat-model-smart-home.json](https://github.com/user-attachments/files/26403475/threat-model-smart-home.json)
<img width="1441" height="596" alt="Screenshot 2026-04-01 161333" src="https://github.com/user-attachments/assets/d0b53fb6-b2df-4d3d-84d5-b39d6deb7334" />
