# **🛡️ Enterprise SOC & Detection Engineering Home Lab**

## **📖 Executive Summary**

This project details the design and implementation of a virtualized **Security Operations Center (SOC)** environment. Unlike standard "flat" home labs, this setup mimics a real-world enterprise architecture using **Network Segmentation** via a pfSense firewall to isolate offensive, defensive, and vulnerable assets.

The primary goal was to simulate the full lifecycle of a cyber attack: from **Infrastructure Setup** and **Red Team Exploitation** to **Blue Team Detection** and **Log Analysis**.

## ---

**🏗️ Network Architecture**

The lab is hosted on **VMware Workstation Pro**, utilizing virtual switches (VMnets) to create four isolated subnets routed strictly through a central firewall.

### **Topology Diagram**
![network_topology](https://github.com/user-attachments/assets/78cad8d0-a34f-43d9-b942-2aab50dec2b3)


| Zone | Subnet | Gateway | Component | IP Address | Role |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **WAN** | N/A | DHCP | **pfSense (WAN)** | 192.168.1.179 | Edge Connectivity |
| **Attacker** | 10.0.1.0/24 | 10.0.1.1 | **Kali Linux** | 10.0.1.2 | Red Team Operations |
| **Monitoring** | 10.0.2.0/24 | 10.0.2.1 | **Wazuh SIEM** | 10.0.2.10 | Log Aggregation & Analysis |
| **Corp Domain** | 10.0.3.0/24 | 10.0.3.1 | **Windows Server 2022** | 10.0.3.10 | Domain Controller (corp.homelab) |
|  |  |  | **Windows 11** | 10.0.3.50 | Domain User Workstation |
| **Vulnerable** | 10.0.4.0/24 | 10.0.4.1 | **Metasploitable 2** | 10.0.4.10 | Intentional Vulnerable Target |

## ---

**🛠️ Technologies & Tools**

* **Network Defense:** pfSense Firewall, Suricata (IDS/IPS).  
* **SIEM & Telemetry:** Wazuh Manager (v4.7), Wazuh Agents, Sysmon.  
* **Identity:** Windows Active Directory Domain Services (AD DS).  
* **Offensive Tools:** NetExec (formerly CrackMapExec), Nmap, Metasploit, Hydra.

## ---

**⚔️ Scenarios & Detection Engineering**

This section documents the specific attack vectors tested and the corresponding detection logic implemented.

### **1\. Network Reconnaissance (Nmap Stealth Scan)**

**Objective:** Detect an attacker attempting to map the network structure using stealth techniques.

* **Attack Command:** nmap \-sS \-p- 10.0.3.10 (Kali Linux)  
* **Defense Mechanism:** Suricata IDS running on pfSense.  
* **Configuration:**  
  * Created a custom Suricata rule to flag rapid SYN packets without accompanying ACK.  
  * Forwarded eve.json logs from pfSense to Wazuh.  
* **Result:** High-Severity Alert triggered in Wazuh Dashboard: *"ET SCAN Nmap Scripting Engine User-Agent Detected"*.

### **2\. Identity Attacks (SMB Brute Force)**

**Objective:** Detect a dictionary attack against the Domain Administrator account.

* **Attack Command:** netexec smb 10.0.3.10 \-u Administrator \-p rockyou.txt \--ignore-pw-decoding  
* **Technical Challenge:** Encountered Python UnicodeDecodeError with the rockyou.txt wordlist. Resolved by using the \--ignore-pw-decoding flag in NetExec.  
* **Telemetry:**  
  * **Windows Security Log:** Spike in **Event ID 4625** (An account failed to log on).  
  * **Logon Type:** 3 (Network).  
  * **Status Code:** 0xC000006D (Bad Password).  
* **Result:** Wazuh correlated 169+ failure events into a single "Brute Force" alert.

### **3\. Credential Dumping (Mimikatz)**

**Objective:** Detect an attempt to extract plaintext passwords or hashes from memory.

* **Attack Command:** sekurlsa::logonpasswords  
* **Defense Mechanism:** Sysmon \+ Wazuh.  
* **Configuration:**  
  * Installed **Sysmon** on the Domain Controller.  
  * Configured sysmonconfig.xml to log **Event ID 10** (Process Access).  
* **Result:** Detected lsass.exe being accessed by mimikatz.exe with GrantedAccess rights indicative of memory reading. Maps to **MITRE ATT\&CK T1003.001**.

## ---

**📸 Visuals & Proof of Concept**

### **The "Vertical Wall" of Alerts**

<img width="1736" height="932" alt="Screenshot 2025-12-15 163316" src="https://github.com/user-attachments/assets/b4ecf659-75d5-403a-b980-43cd7f1bdf79" />


*Visualizing the brute force attack volume in real-time.*

### **Suricata Interface on pfSense**
<img width="1643" height="920" alt="Screenshot 2025-12-15 163458" src="https://github.com/user-attachments/assets/835aaef1-1763-4b6d-a132-94a4f4c2b9bf" />

*Managing interface assignments and IDS rules.*

## ---

**🚀 Lessons Learned & Future Work**

1. **Protocol Decoding:** Learned the importance of UTF-8 encoding handling in Python-based offensive tools (NetExec).  
2. **Noise vs. Signal:** A brute force attack generates massive log volume; tuning SIEM rules to alert on *frequency* rather than *single events* is critical to avoid fatigue.  
3. **Active Response:** Future iterations will implement Wazuh's "Active Response" module to automatically block the attacker's IP at the pfSense firewall level upon detection.

### ---

**👤 Author**

* ---

  LinkedIn Profile Link\](https://www.linkedin.com/in/gokulsathiyamurthy088/
