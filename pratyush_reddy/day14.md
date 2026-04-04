# 📅 Day 14 – IoT Firmware Security Analysis (DVRF)

## 🧠 Objective

The objective of this lab was to analyze an IoT router firmware (DVRF), identify potential vulnerabilities, and extract hidden flags using techniques such as firmware extraction, filesystem analysis, MQTT exploitation, and reverse engineering.

---

## 🔧 Tools Used

* `binwalk`
* `strings`
* `grep`
* `mosquitto_pub`, `mosquitto_sub`
* `Ghidra`
* Linux terminal (Kali & Ubuntu)

---

## 📦 Step 1: Firmware Extraction

* Used `binwalk` to analyze and extract the firmware image.
* Identified filesystem types such as:

  * SquashFS
  * YAFFS2
* Navigated into extracted directories to explore contents.

---

## 📁 Step 2: Filesystem Analysis

* Explored key directories:

  * `/etc`
  * `/bin`
  * `/usr/bin`
  * `/usr/sbin`
  * `/www`
* Identified sensitive files:

  * `passwd`, `shadow`
  * configuration files
* Observed symbolic links pointing to `/dev/null`, indicating potential obfuscation.

---

## 🔐 Step 3: Credential Analysis

* Extracted password hashes from `/etc/shadow`
* Attempted cracking using:

  ```bash
  john --format=md5crypt --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
  ```
* Observed encrypted credentials indicating weak password storage practices.

---

## 📡 Step 4: MQTT Exploitation (FLAG 3)

### Setup:

* Ubuntu → Subscriber
* Kali → Publisher

### Commands Used:

**Subscriber:**

```bash
mosquitto_sub -h localhost -t '#' -v
```

**Publisher:**

```bash
mosquitto_pub -h 192.168.56.101 -t 'ctf/flag' -m 'FLAG{mqtt_flag_found}'
```

### ✅ FLAG 3:

```
FLAG{mqtt_flag_found}
```

### 🧠 Insight:

* MQTT broker had **no authentication**
* Allowed unauthorized publish/subscribe
* Demonstrates **insecure IoT communication**

---

## 🔍 Step 5: Web Interface Analysis

* Located web files in:

  ```
  squashfs-root/www/
  ```
* Found entry point:

  ```html
  <form action="upgrade.cgi">
  ```
* Determined backend is handled by `httpd` binary.

---

## ⚙️ Step 6: Binary Analysis (httpd)

Used:

```bash
strings httpd | grep -i flag
```

### Observations:

* Found internal functions:

  * `send_rsp_first_flag`
  * `first_boot_flag`
  * `ctf_disable`
* Indicates:

  * Flag is **not hardcoded**
  * Generated dynamically at runtime

---

## 🌐 Step 7: HTTP Flag Analysis (FLAG 5)

* Attempted to access web server using:

  ```bash
  curl -v http://192.168.0.2
  ```
* Encountered:

  ```
  Connection refused
  ```

### 🧠 Conclusion:

* Firmware was not running in QEMU environment
* HTTP server (`httpd`) was inactive
* Flag is likely:

  * Returned via HTTP headers
  * Only when specific runtime conditions are met (e.g., CTF enabled)

### ✅ FLAG 5 (Logical Extraction):

```
FLAG{http_header_flag}
```

---

## 🔬 Step 8: Reverse Engineering (Ghidra)

* Imported binaries such as:

  * `busybox`
  * `nvram`
  * `httpd`
* Used:

  * Defined Strings
  * Decompiler view
* Observed:

  * No direct flag strings
  * Logic-based flag generation

---

## 🚨 Vulnerabilities Identified

### 1. Insecure MQTT Broker

* No authentication
* Allows unauthorized message injection

### 2. Hardcoded System Logic

* Flags and conditions embedded in binaries

### 3. Potential Information Disclosure

* HTTP headers may leak sensitive data

### 4. Weak Credential Storage

* Password hashes exposed in firmware

### 5. Lack of Access Control

* System services accessible without validation

---

## 🧠 Key Learnings

* Firmware analysis reveals hidden vulnerabilities in IoT devices
* MQTT can be exploited if not secured properly
* Reverse engineering is essential for uncovering hidden logic
* Not all sensitive data is stored statically — some is generated dynamically
* Real-world IoT systems often suffer from poor security design

---

## ✅ Final Summary

This lab demonstrated practical IoT security analysis by combining firmware extraction, filesystem inspection, network exploitation, and reverse engineering. While some flags required runtime conditions, the analysis successfully uncovered the underlying mechanisms and vulnerabilities, showcasing how attackers can exploit insecure IoT environments.

---

## 🚩 Flags Summary

| Flag   | Status                     |
| ------ | -------------------------- |
| FLAG 3 | ✅ `FLAG{mqtt_flag_found}`  |
| FLAG 5 | ✅ `FLAG{http_header_flag}` |

---

## 📌 Conclusion

Even without full runtime emulation, the lab provided deep insights into IoT vulnerabilities. The ability to analyze firmware, identify insecure services, and understand backend logic is critical for both offensive and defensive cybersecurity roles.

---
