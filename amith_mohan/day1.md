## -->Day 1 

## -->Overview
Day 1 focused on understanding the basics of IoT security, setting up the lab environment using Kali Linux, and exploring essential security tools.

---------------------------------------------------------------

## -->Lab Setup

- Installed VirtualBox on my system
- Imported Kali Linux Virtual Machine
- Successfully booted Kali Linux
- Logged into Kali using default credentials
- Explored the Kali Linux interface and menu system

---------------------------------------------------------------

## -->Tools Verification

The following tools were verified in the terminal:

- binwalk  ✔
- nmap --version ✔
- wireshark --version ✔

All tools returned valid version outputs, confirming successful setup.

---------------------------------------------------------------

## -->OWASP IoT Top 10

The **OWASP IoT Top 10** highlights the major security risks present in IoT devices due to weak design and poor security practices. Each vulnerability represents a different way in which attackers can exploit these devices.

### 1. Weak, guessable, or hardcoded passwords
Devices often use default or predictable credentials. Attackers can scan and log in to thousands of devices, as seen in the Mirai botnet attack.

### 2. Insecure network services
Unnecessary services like Telnet or FTP run on devices with weak security, allowing attackers to gain remote access easily.

### 3. Insecure ecosystem interfaces
Mobile apps, web dashboards, and APIs may lack proper authentication or encryption, allowing attackers to access devices or user data.

### 4. Lack of secure update mechanisms
Devices do not properly verify firmware updates, allowing attackers to install malicious updates and take full control.

### 5. Use of insecure or outdated components
Devices rely on outdated libraries with known vulnerabilities, exposing them to attacks.

### 6. Insufficient privacy protection
Sensitive data like audio, video, or location is collected without proper protection, leading to privacy risks.

### 7. Insecure data transfer and storage
Data is transmitted or stored without encryption, making it easy for attackers to intercept or steal information.

### 8. Lack of device management
Without proper monitoring and updates, compromised devices remain undetected and can be used in large-scale attacks.

### 9. Insecure default settings
Devices are shipped with unsafe configurations like open ports or enabled remote access, which attackers exploit easily.

### 10. Lack of physical hardening
Attackers can physically access devices, extract firmware, and reverse engineer them to find vulnerabilities.

---------------------------------------------------------------

## -->AI Task Summary

Used Google Gemini,chatgpt to understand OWASP IoT Top 10 with real-world examples.  
Rewrote the concepts in my own words for better understanding.

---------------------------------------------------------------

## -->Kali Linux Exploration

Explored a tool in the Kali Linux menu:


- nmap (network scanning tool used to discover devices, open ports, and services on a network)
- and few other categories

---------------------------------------------------------------

## -->Files Downloaded

- IoTGoat firmware image (.img)
- DVRF vulnerable firmware (.zip)

These will be used in upcoming labs for firmware analysis and exploitation.


---------------------------------------------------------------

## -->Outcome

- Successfully set up Kali Linux virtual lab
- Verified essential security tools
- Understood OWASP IoT Top 10 vulnerabilities
- Explored Kali Linux tools and categories
- Prepared environment for further IoT security analysis

---------------------------------------------------------------
