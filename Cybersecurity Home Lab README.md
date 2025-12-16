# **🛡️ Enterprise SOC & Detection Engineering Home Lab**

\!([https://img.shields.io/badge/Security-Blue%20Team-blue)\!(https://img.shields.io/badge/Platform-VMware%20%7C%20pfSense-orange)\!(https://img.shields.io/badge/SIEM-Wazuh-green](https://img.shields.io/badge/Security-Blue%20Team-blue)\!([https://img.shields.io/badge/Platform-VMware%20%7C%20pfSense-orange)\!(https://img.shields.io/badge/SIEM-Wazuh-green](https://img.shields.io/badge/Platform-VMware%20%7C%20pfSense-orange)\!(https://img.shields.io/badge/SIEM-Wazuh-green)))

## **📖 Executive Summary**

This project details the design and implementation of a virtualized **Security Operations Center (SOC)** environment. Unlike standard "flat" home labs, this setup mimics a real-world enterprise architecture using **Network Segmentation** via a pfSense firewall to isolate offensive, defensive, and vulnerable assets.

The primary goal was to simulate the full lifecycle of a cyber attack: from **Infrastructure Setup** and **Red Team Exploitation** to **Blue Team Detection** and **Log Analysis**.

## ---

**🏗️ Network Architecture**

The lab is hosted on **VMware Workstation Pro**, utilizing virtual switches (VMnets) to create four isolated subnets routed strictly through a central firewall.

### **Topology Diagram**

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1200 900" width="1200" height="900">
  <!-- Background -->
  <rect width="1200" height="900" fill="#0d1117"/>
  
  <!-- Title -->
  <text x="600" y="40" font-family="Arial, sans-serif" font-size="28" font-weight="bold" fill="#58a6ff" text-anchor="middle">
    Cybersecurity Lab Network Topology
  </text>
  
  <!-- WAN Zone -->
  <g id="wan-zone">
    <rect x="450" y="80" width="300" height="100" rx="10" fill="#161b22" stroke="#58a6ff" stroke-width="2"/>
    <text x="600" y="105" font-family="Arial, sans-serif" font-size="14" font-weight="bold" fill="#58a6ff" text-anchor="middle">
      🌐 WAN ZONE
    </text>
    <text x="600" y="125" font-family="Arial, sans-serif" font-size="11" fill="#8b949e" text-anchor="middle">
      Gateway: DHCP
    </text>
    <rect x="470" y="135" width="260" height="35" rx="5" fill="#0d1117" stroke="#58a6ff" stroke-width="1.5"/>
    <text x="600" y="153" font-family="Arial, sans-serif" font-size="12" font-weight="bold" fill="#58a6ff" text-anchor="middle">
      pfSense (WAN)
    </text>
    <text x="600" y="167" font-family="Courier New, monospace" font-size="10" fill="#8b949e" text-anchor="middle">
      192.168.1.179 | Edge Connectivity
    </text>
  </g>
  
  <!-- Attacker Zone -->
  <g id="attacker-zone">
    <rect x="50" y="250" width="280" height="120" rx="10" fill="#161b22" stroke="#f85149" stroke-width="2"/>
    <text x="190" y="275" font-family="Arial, sans-serif" font-size="14" font-weight="bold" fill="#f85149" text-anchor="middle">
      🎯 ATTACKER ZONE
    </text>
    <text x="190" y="295" font-family="Arial, sans-serif" font-size="11" fill="#8b949e" text-anchor="middle">
      10.0.1.0/24 | GW: 10.0.1.1
    </text>
    <rect x="70" y="305" width="240" height="55" rx="5" fill="#0d1117" stroke="#f85149" stroke-width="1.5"/>
    <text x="190" y="325" font-family="Arial, sans-serif" font-size="12" font-weight="bold" fill="#f85149" text-anchor="middle">
      Kali Linux
    </text>
    <text x="190" y="340" font-family="Courier New, monospace" font-size="10" fill="#8b949e" text-anchor="middle">
      10.0.1.2
    </text>
    <text x="190" y="353" font-family="Arial, sans-serif" font-size="9" fill="#6e7681" text-anchor="middle">
      Red Team Operations
    </text>
  </g>
  
  <!-- Monitoring Zone -->
  <g id="monitoring-zone">
    <rect x="870" y="250" width="280" height="120" rx="10" fill="#161b22" stroke="#f0883e" stroke-width="2"/>
    <text x="1010" y="275" font-family="Arial, sans-serif" font-size="14" font-weight="bold" fill="#f0883e" text-anchor="middle">
      📊 MONITORING ZONE
    </text>
    <text x="1010" y="295" font-family="Arial, sans-serif" font-size="11" fill="#8b949e" text-anchor="middle">
      10.0.2.0/24 | GW: 10.0.2.1
    </text>
    <rect x="890" y="305" width="240" height="55" rx="5" fill="#0d1117" stroke="#f0883e" stroke-width="1.5"/>
    <text x="1010" y="325" font-family="Arial, sans-serif" font-size="12" font-weight="bold" fill="#f0883e" text-anchor="middle">
      Wazuh SIEM
    </text>
    <text x="1010" y="340" font-family="Courier New, monospace" font-size="10" fill="#8b949e" text-anchor="middle">
      10.0.2.10
    </text>
    <text x="1010" y="353" font-family="Arial, sans-serif" font-size="9" fill="#6e7681" text-anchor="middle">
      Log Aggregation &amp; Analysis
    </text>
  </g>
  
  <!-- Corp Domain Zone -->
  <g id="corp-zone">
    <rect x="420" y="450" width="360" height="200" rx="10" fill="#161b22" stroke="#3fb950" stroke-width="2"/>
    <text x="600" y="475" font-family="Arial, sans-serif" font-size="14" font-weight="bold" fill="#3fb950" text-anchor="middle">
      🏢 CORP DOMAIN ZONE
    </text>
    <text x="600" y="495" font-family="Arial, sans-serif" font-size="11" fill="#8b949e" text-anchor="middle">
      10.0.3.0/24 | GW: 10.0.3.1
    </text>
    <!-- Windows Server -->
    <rect x="440" y="505" width="320" height="60" rx="5" fill="#0d1117" stroke="#3fb950" stroke-width="1.5"/>
    <text x="600" y="525" font-family="Arial, sans-serif" font-size="12" font-weight="bold" fill="#3fb950" text-anchor="middle">
      Windows Server 2022
    </text>
    <text x="600" y="540" font-family="Courier New, monospace" font-size="10" fill="#8b949e" text-anchor="middle">
      10.0.3.10
    </text>
    <text x="600" y="555" font-family="Arial, sans-serif" font-size="9" fill="#6e7681" text-anchor="middle">
      Domain Controller (corp.homelab)
    </text>
    <!-- Windows 11 -->
    <rect x="440" y="575" width="320" height="60" rx="5" fill="#0d1117" stroke="#3fb950" stroke-width="1.5"/>
    <text x="600" y="595" font-family="Arial, sans-serif" font-size="12" font-weight="bold" fill="#3fb950" text-anchor="middle">
      Windows 11
    </text>
    <text x="600" y="610" font-family="Courier New, monospace" font-size="10" fill="#8b949e" text-anchor="middle">
      10.0.3.50
    </text>
    <text x="600" y="625" font-family="Arial, sans-serif" font-size="9" fill="#6e7681" text-anchor="middle">
      Domain User Workstation
    </text>
  </g>
  
  <!-- Vulnerable Zone -->
  <g id="vulnerable-zone">
    <rect x="450" y="720" width="300" height="120" rx="10" fill="#161b22" stroke="#bc8cff" stroke-width="2"/>
    <text x="600" y="745" font-family="Arial, sans-serif" font-size="14" font-weight="bold" fill="#bc8cff" text-anchor="middle">
      ⚠️ VULNERABLE ZONE
    </text>
    <text x="600" y="765" font-family="Arial, sans-serif" font-size="11" fill="#8b949e" text-anchor="middle">
      10.0.4.0/24 | GW: 10.0.4.1
    </text>
    <rect x="470" y="775" width="260" height="55" rx="5" fill="#0d1117" stroke="#bc8cff" stroke-width="1.5"/>
    <text x="600" y="795" font-family="Arial, sans-serif" font-size="12" font-weight="bold" fill="#bc8cff" text-anchor="middle">
      Metasploitable 2
    </text>
    <text x="600" y="810" font-family="Courier New, monospace" font-size="10" fill="#8b949e" text-anchor="middle">
      10.0.4.10
    </text>
    <text x="600" y="823" font-family="Arial, sans-serif" font-size="9" fill="#6e7681" text-anchor="middle">
      Intentional Vulnerable Target
    </text>
  </g>
  
  <!-- Connection Lines -->
  <!-- WAN to Attacker -->
  <line x1="520" y1="180" x2="280" y2="250" stroke="#f85149" stroke-width="2" stroke-dasharray="5,5" opacity="0.5"/>
  
  <!-- WAN to Monitoring -->
  <line x1="680" y1="180" x2="920" y2="250" stroke="#f0883e" stroke-width="2" stroke-dasharray="5,5" opacity="0.5"/>
  
  <!-- WAN to Corp -->
  <line x1="600" y1="180" x2="600" y2="450" stroke="#3fb950" stroke-width="2" stroke-dasharray="5,5" opacity="0.5"/>
  
  <!-- WAN to Vulnerable -->
  <line x1="600" y1="180" x2="600" y2="720" stroke="#bc8cff" stroke-width="2" stroke-dasharray="5,5" opacity="0.3"/>
  
  <!-- Attacker to Corp -->
  <line x1="330" y1="340" x2="420" y2="520" stroke="#f85149" stroke-width="2" stroke-dasharray="5,5" opacity="0.4"/>
  
  <!-- Attacker to Vulnerable -->
  <line x1="300" y1="370" x2="450" y2="760" stroke="#f85149" stroke-width="2" stroke-dasharray="5,5" opacity="0.4"/>
  
  <!-- Monitoring to Corp -->
  <line x1="870" y1="340" x2="780" y2="520" stroke="#f0883e" stroke-width="2" stroke-dasharray="5,5" opacity="0.4"/>
  
  <!-- Monitoring to Vulnerable -->
  <line x1="920" y1="370" x2="750" y2="760" stroke="#f0883e" stroke-width="2" stroke-dasharray="5,5" opacity="0.4"/>
  
  <!-- Legend -->
  <g id="legend">
    <rect x="20" y="820" width="220" height="65" rx="5" fill="#161b22" stroke="#30363d" stroke-width="1"/>
    <text x="30" y="840" font-family="Arial, sans-serif" font-size="11" font-weight="bold" fill="#c9d1d9">
      Network Flow Legend:
    </text>
    <line x1="30" y1="850" x2="60" y2="850" stroke="#58a6ff" stroke-width="2" stroke-dasharray="5,5"/>
    <text x="70" y="854" font-family="Arial, sans-serif" font-size="10" fill="#8b949e">
      Internet Gateway
    </text>
    <line x1="30" y1="865" x2="60" y2="865" stroke="#f85149" stroke-width="2" stroke-dasharray="5,5"/>
    <text x="70" y="869" font-family="Arial, sans-serif" font-size="10" fill="#8b949e">
      Attack Path
    </text>
    <line x1="30" y1="880" x2="60" y2="880" stroke="#f0883e" stroke-width="2" stroke-dasharray="5,5"/>
    <text x="70" y="884" font-family="Arial, sans-serif" font-size="10" fill="#8b949e">
      Monitoring Flow
    </text>
  </g>
</svg>

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
