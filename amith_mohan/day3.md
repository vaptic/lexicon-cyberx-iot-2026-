# Day 3 
**Date:** 2026-03-23  

---

## Overview

This lab focuses on understanding how MQTT behaves in a real environment, how its traffic looks on the network, and how its security compares with HTTPS. It also demonstrates how misconfigurations can lead to serious data exposure.

---

## MQTT vs HTTPS — Security Comparison

Both MQTT and HTTPS can be secured using TLS, but they differ in how security is applied in practice.

### Encryption Behavior

- **HTTPS**
  - Always runs over TLS by default  
  - Ensures encryption, integrity, and server verification automatically  

- **MQTT**
  - Operates without encryption unless configured  
  - Default port (1883) sends data in plaintext  
  - Secure version (8883) must be explicitly enabled  

---

### Authentication Approach

- **HTTPS**
  - Uses certificate-based server authentication  
  - Supports tokens, API keys, OAuth, and sessions  
  - Widely standardized and integrated  

- **MQTT**
  - Supports username/password and certificate-based authentication  
  - Authentication is handled by the broker  
  - Implementation varies across deployments  

---

### Authorization Control

- **HTTPS**
  - Managed at application level (API endpoints, roles, tokens)  
  - Fine-grained control over resources  

- **MQTT**
  - Uses topic-based permissions (ACLs)  
  - Controls which topics a client can publish or subscribe to  
  - Requires careful broker configuration  

---

### Security Ecosystem

- **HTTPS**
  - Backed by mature tools and frameworks  
  - Easier to audit and manage securely  

- **MQTT**
  - Cryptography can be strong, but setup is manual  
  - Many real-world systems remain insecure due to misconfiguration  

---

### IoT Usage Perspective

- **HTTPS**
  - Short-lived connections  
  - Higher overhead  
  - Suitable for firmware updates and APIs  

- **MQTT**
  - Persistent connections  
  - Lightweight and efficient for continuous data  
  - Risky if compromised due to ongoing access  

---

## Key Insight

> MQTT is efficient but requires proper configuration for security,  
> while HTTPS is secure by default but heavier for constrained devices.

---

# MQTT Wildcard Subscription Attack

This attack leverages MQTT’s wildcard feature to gain access to a large set of messages from a broker.

---

## Concept

MQTT topics support wildcards:

- `+` → matches a single level  
- `#` → matches all levels  

Subscribing to `#` allows a client to receive every message published on the broker.

---

## Attack Flow

1. Identify an exposed MQTT broker  
2. Connect without authentication (if allowed)  
3. Subscribe using:
   ```bash
   #
