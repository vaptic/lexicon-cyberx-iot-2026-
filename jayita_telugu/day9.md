# Day 9 - IoT Device Discovery: Nmap & Shodan

## Lab Setup
- Kali Linux VM (Attacker) - 192.168.56.101
- Ubuntu IoT Target VM - 192.168.56.10

## Nmap Scans

### Network Discovery
`nmap -sn 192.168.56.0/24`
Found Ubuntu VM live at 192.168.56.10

### Quick Port Scan
`nmap -F 192.168.56.10`
Found open ports including 22 (SSH) and 1883 (MQTT)

### Service Version Scan
`nmap -sV -p 1-1000 192.168.56.10`
Identified Mosquitto MQTT broker version running on port 1883

### Aggressive Scan
`nmap -A 192.168.56.10 > nmap-full-scan.txt`
Full OS detection, service versions, and traceroute completed

### MQTT Script
`nmap --script mqtt-subscribe 192.168.56.10 -p 1883`
Successfully connected to MQTT broker and retrieved topic information

## Shodan Findings
- Searched `product:Mosquitto country:IN` - found hundreds of exposed MQTT brokers in India with no authentication
- Searched `default password port:80 country:IN` - found numerous devices still using factory default credentials

## AI Task - Nmap NSE Scripts for IoT
Perplexity AI recommended these NSE scripts for IoT fingerprinting:
- `nmap --script mqtt-subscribe` - subscribes to MQTT broker and lists topics
- `nmap --script http-default-accounts` - tests default credentials on web interfaces
- `nmap --script banner` - grabs service banners revealing device model and firmware
- `nmap --script snmp-brute` - tests common SNMP community strings on IoT devices

Ran `mqtt-subscribe` script against Ubuntu VM at 192.168.56.10 - successfully connected to broker on port 1883.
<img width="1208" height="3424" alt="www shodan io_search_query=product3AIN" src="https://github.com/user-attachments/assets/1802c426-235e-46cc-ba30-07e541f9191e" />
<img width="1208" height="3088" alt="www shodan io_search_query=default+password+port3AIN" src="https://github.com/user-attachments/assets/4710deed-df81-49a9-820c-ac21f2172d04" />
.
