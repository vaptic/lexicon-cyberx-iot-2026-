# IoT Device Discovery: Nmap & Shodan


## Lab Environment

| Role | OS | IP Address |
|---|---|---|
| Attacker | Kali Linux | 192.168.56.102 |
| IoT Target | Ubuntu Server 24.04 LTS | 192.168.56.101 |
| Tool 1 | Nmap 7.95 | Pre-installed on Kali |
| Tool 2 | Shodan.io | Free account |

---

## Theory Summary

### Reconnaissance — The First Stage of Any Attack
Before attacking anything, an attacker (and a defender) needs to know what is on the network. Nmap can identify:
- What devices are on the network
- What ports are open
- What services are running
- What software version and firmware is in use

### Key Nmap Flags for IoT
| Flag | Purpose |
|---|---|
| `-sn` | Ping sweep — discover live hosts, no port scan |
| `-F` | Fast scan — top 100 ports only |
| `-sV` | Service version detection |
| `-O` | OS detection |
| `-A` | Aggressive — enables OS, version, scripts, traceroute |
| `-p-` | Scan all 65535 ports |
| `--script` | Run NSE (Nmap Scripting Engine) scripts |

### What is Shodan?
Shodan is a search engine for internet-connected devices. Unlike Google which indexes web pages, Shodan indexes everything — routers, cameras, industrial controllers, MQTT brokers, and medical devices. It reveals publicly exposed devices without any active scanning on your part.

---

## Step-by-Step Lab

### Step 1 — Network Discovery Scan
```bash
nmap -sn 192.168.56.0/24
```

**Output:**
```
Starting Nmap 7.95 at 2026-03-31 12:28 IST
Nmap scan report for 192.168.56.1
Host is up (0.0011s latency).
MAC Address: 0A:00:27:00:00:15 (Unknown)

Nmap scan report for 192.168.56.100
Host is up.
MAC Address: 08:00:27:BE:B5:2B (PCS Systemtechnik/Oracle VirtualBox)

Nmap scan report for 192.168.56.101
Host is up (0.00071s latency).
MAC Address: 08:00:27:29:9B:3F (PCS Systemtechnik/Oracle VirtualBox)

Nmap scan report for 192.168.56.102
Host is up.

Nmap done: 256 IP addresses (4 hosts up) scanned in 27.96 seconds
```

**Hosts Discovered:**
| IP | Role |
|---|---|
| 192.168.56.1 | VirtualBox host gateway |
| 192.168.56.100 | VirtualBox DHCP server |
| 192.168.56.101 | **Ubuntu IoT target** |
| 192.168.56.102 | **Kali attacker machine** |

---

### Step 2 — Fast Port Scan
```bash
nmap -F 192.168.56.101
```

**Output:**
```
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 08:00:27:29:9B:3F (Oracle VirtualBox)

Nmap done: 1 IP address (1 host up) scanned in 13.22 seconds
```

Only port 22 (SSH) visible in the top 100 ports. Port 1883 (MQTT) is above this range and was not scanned here.

---

### Step 3 — Full Service Version Scan (ports 1–2000)
```bash
nmap -sV -p 1-2000 192.168.56.101
```

**Output:**
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
1883/tcp open  mosquitto version 2.0.18

MAC Address: 08:00:27:29:9B:3F (PCS Systemtechnik/Oracle VirtualBox)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed.
Nmap done: 1 IP address (1 host up) scanned in 26.51 seconds
```

**Services Identified:**
| Port | State | Service | Version |
|---|---|---|---|
| 22/tcp | open | SSH | OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 |
| 1883/tcp | open | MQTT | Mosquitto 2.0.18 |

> **Security Note:** With exact version numbers, an attacker can immediately search CVE databases for known vulnerabilities in OpenSSH 9.6p1 and Mosquitto 2.0.18 — without ever logging in.

---

### Step 4 — Aggressive Full Scan
```bash
nmap -A -p 22,1883 192.168.56.101 > nmap-full-scan.txt
cat nmap-full-scan.txt
```
Saved to `findings/nmap-full-scan.txt`. Reveals OS fingerprint, traceroute, and detailed service banners.

---

### Step 5 — MQTT NSE Script
```bash
nmap --script mqtt-subscribe -p 1883 192.168.56.101
```
Attempted to subscribe to the MQTT broker and pull live messages from the broker anonymously.

---

## Shodan Research

### Search 1 — Global Mosquitto Brokers
**Query:** `mosquitto`

| Metric | Value |
|---|---|
| Total Results | **201** |
| Top Country | United States (55) |
| 2nd | Germany (22) |
| 3rd | China (13) |
| 4th | Hong Kong (10) |
| 5th | Singapore (9) |

**Notable Finding:**  
One result (`138.68.124.70`, DigitalOcean, Germany) showed live MQTT topics being broadcast publicly:
```
$SYS/broker/version
$SYS/broker/connection/mosquitto-5869b85d6b-6qm2s.mqtt_bridge/state
$SYS/broker/connection/fa1-zone-ad1.TrafalgarSquare/state
$SYS/broker/connection/ip-172-31-36-94.PearlStreet/state
```
This broker is bridged to multiple other brokers — all of which are now exposed. `MQTT Connection Code: 0` means anonymous access is allowed.

---

### Search 2 — Default Password Devices in India
**Query:** `default password port:80 country:IN`

| Metric | Value |
|---|---|
| Total Results | **12** |
| Top City | Pune (5) |
| 2nd | Mumbai (4) |
| Organisations | Microsoft, Amazon, Atria Convergence |

**Notable Finding:**  
An **HP Ink Tank Wireless 410 printer** in Quthbullapur was fully exposed — including its model number (`Z6Z97A`) and serial number (`CN88E3G48T06V1`). This device's web interface is publicly accessible with default credentials.

---

### Search 3 — Exposed MQTT Brokers in India
**Query:** `product:Mosquitto country:IN`

| Metric | Value |
|---|---|
| Total Results | **7,044** |
| Top City | New Delhi (1,349) |
| 2nd | Mumbai (1,302) |
| 3rd | Delhi (1,118) |
| 4th | Kolkata (549) |
| 5th | Gurugram (498) |
| Top ISP | BHARTI-AIRTEL (1,957) |

**All results showed `MQTT Connection Code: 0`** — meaning every single one of these 7,044 brokers allows anonymous connections, just like our lab Ubuntu VM.

---

## Attacker Reconnaissance Comparison
### What Can Be Learned Using ONLY Nmap + Shodan — Before Touching a Device?

### From Nmap (Active — requires knowing the IP)

| Information | Command | Finding |
|---|---|---|
| Live devices on network | `nmap -sn 192.168.56.0/24` | 4 hosts, Ubuntu at .101 |
| Open ports | `nmap -F 192.168.56.101` | Port 22 (SSH) |
| Exact software & version | `nmap -sV -p 1-2000` | OpenSSH 9.6p1, Mosquitto 2.0.18 |
| OS & kernel | `nmap -A` | Linux, CPE: linux_kernel |
| MQTT access | `nmap --script mqtt-subscribe` | Anonymous access confirmed |

### From Shodan (Passive — no target interaction at all)

| Information | Query | Finding |
|---|---|---|
| Exposed MQTT brokers in India | `product:Mosquitto country:IN` | 7,044 brokers, all anonymous |
| Default credential devices | `default password port:80 country:IN` | 12 devices including printer with serial number |
| Live MQTT data streams | `mosquitto` | Topics and bridge connections publicly visible |
| Physical location | Any IP lookup | City, ISP, organisation name |

### Combined Attacker Picture

```
Target: 192.168.56.101 (or any of the 7,044 Indian MQTT brokers)

From Nmap:                           From Shodan:
├── OS: Linux                        ├── ISP name
├── SSH: OpenSSH 9.6p1              ├── City & Country
├── MQTT: Mosquitto 2.0.18          ├── Organisation name
├── MQTT: Anonymous access ON       ├── First seen / Last seen dates
├── MQTT: Port 1883 open            ├── All historically open ports
└── MQTT: Live broker confirmed     └── Known CVEs for that version
```

**Before sending a single malicious packet, the attacker knows:**
- ✅ Exact software versions → searchable in CVE databases
- ✅ Anonymous MQTT → can connect and read/publish any message
- ✅ SSH is open → can attempt credential brute force
- ✅ Physical location → jurisdiction and monitoring awareness
- ✅ ISP → can gauge likelihood of traffic monitoring

### Key Takeaway
> Nmap + Shodan together give an attacker a **complete reconnaissance picture** — device type, software, versions, location, open services, and live data — in under 10 minutes, with **no authentication and no direct interaction with the target.**  
> This is why **security by obscurity fails** — simply not advertising your device is not enough if it is internet-connected.

---

## AI Tool Task

**Tool used:** Perplexity AI  
**Prompt:** *"What are the most effective Nmap NSE scripts for IoT device fingerprinting and vulnerability detection? Give examples with command syntax."*

**Scripts Recommended by Perplexity & Applied:**

| Script | Command | Purpose |
|---|---|---|
| `banner` | `nmap --script banner -p 22,1883 192.168.56.101` | Grab service banners revealing device info |
| `mqtt-subscribe` | `nmap --script mqtt-subscribe -p 1883 192.168.56.101` | Subscribe to MQTT and read live messages |
| `http-default-accounts` | `nmap --script http-default-accounts 192.168.56.101` | Test for default credentials on web interfaces |
| `ssh-auth-methods` | `nmap --script ssh-auth-methods -p 22 192.168.56.101` | Enumerate SSH authentication methods |

**Result of `mqtt-subscribe` against Ubuntu:**  
The script successfully connected to the broker anonymously and subscribed to topics — confirming that any attacker could read all IoT sensor data without any credentials.

---

## Files
https://github.com/vaptic/lexicon-cyberx-iot-2026-/blob/08146d4fa6755a242e99312d52eb99639206424f/pratyush_reddy/findings/nmap-full-scans.txt
<img width="1361" height="603" alt="Screenshot 2026-03-31 124450" src="https://github.com/user-attachments/assets/e33e0827-3db4-4b15-8be9-dfc45eb6333c" />
<img width="1365" height="602" alt="Screenshot 2026-03-31 124503" src="https://github.com/user-attachments/assets/e356c291-6977-4336-88c3-8a4826245ec3" />
<img width="1365" height="599" alt="Screenshot 2026-03-31 124655" src="https://github.com/user-attachments/assets/e4b465b1-a367-495c-afc5-17a1c3235dc7" />
