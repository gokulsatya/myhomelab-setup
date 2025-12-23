# 🛡️ Day 2: Infrastructure Resilience & Agent Deployment

**Date:** December 22, 2025  
**Role:** SOC Analyst / Security Engineer  
**Status:** ✅ Complete  

---

## 🎯 Objective
After establishing basic network segmentation on Day 1, the goal for Day 2 was to onboard the **Windows Server 2022 (Domain Controller)** into the SIEM. This required configuring strict firewall traversal rules in pfSense and stabilizing the Wazuh Manager, which was suffering from resource exhaustion crashes.

## 🏗️ Architecture Overview
* **Target Asset:** Windows Server 2022 (Domain Controller) `@ 10.0.3.10`
* **Protection:** pfSense Firewall + Windows Defender Firewall
* **Monitoring:** Wazuh SIEM (Ubuntu 24.04) `@ 10.0.2.10`
* **Network Path:** `AD_LAB (VLAN 30)` ↔️ `pfSense Router` ↔️ `MONITOR (VLAN 20)`

---

## 🔧 Incident Log: Troubleshooting "The Black Hole"

### 🔴 Incident 1: The "Silent" Agent (Route Failure)
* **Symptom:** The Windows Agent was installed but appeared as **"Disconnected"** in the Wazuh Dashboard.
* **Logs:** `ossec.log` showed error `1216: Unable to connect to 10.0.2.10:1514`.
* **Investigation:**
    * `Ping 10.0.2.10` from the Server resulted in **"Request Timed Out"**.
    * `Test-NetConnection -Port 1515` returned **False**.
    * `tracert 10.0.2.10` revealed packets were dying at the local gateway (`10.0.3.1`).
* **Resolution:**
    1.  Configured a **Pass Rule** on the pfSense `AD_LAB` interface to allow traffic *into* the router.
    2.  Configured a matching **Pass Rule** on the `MONITOR` interface to allow the *reply* traffic back to the Server.
    3.  **Critical Fix:** Manually clicked **"Apply Changes"** in pfSense to flush the state table (rules do not activate on "Save" alone).

### 🔴 Incident 2: The Service Crash (Resource Exhaustion)
* **Symptom:** During connection testing, the Windows Agent suddenly stopped receiving replies, even though the network path was verified open.
* **Forensics:**
    * Ran `sudo systemctl status wazuh-manager` on the SIEM.
    * Discovered error: `Active: failed (Result: signal) code=killed, signal=KILL`.
* **Diagnosis:** The Linux Kernel's **OOM Killer (Out of Memory)** terminated the Wazuh processes because the VM (4GB RAM) was overwhelmed by running the Indexer (Java) and Dashboard simultaneously.
* **Mitigation:**
    * Implemented a **Sequenced Restart Protocol**: Stopped all services, then started them one-by-one with 30-second delays (`Indexer` → `Manager` → `Dashboard`) to prevent memory spikes during initialization.

### 🔴 Incident 3: The "Zombie" Firewall
* **Symptom:** Windows refused to connect even after the router rules were fixed.
* **Diagnosis:** The **Windows Defender Firewall** had classified the new lab network as "Public" (Untrusted), blocking inbound return traffic from the SIEM.
* **Resolution:**
    * Verified connectivity by temporarily disabling profiles via PowerShell:
        ```powershell
        Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
        ```

---

## 🛠️ Key Skills Demonstrated
| Category | Skill |
| :--- | :--- |
| **Network Forensics** | Packet tracing with `tracert`, `Test-NetConnection`, and `ss -tulnp` |
| **Linux Admin** | Diagnosing Kernel OOM kills & managing `systemd` service dependencies |
| **Firewall Logic** | Configuring stateful packet filtering & handling rule application states |
| **Agent Ops** | Troubleshooting Wazuh agent encryption keys and connectivity logs |

---

## ⏭️ Next Steps (Day 3 Preview)
With the Domain Controller now active and reporting logs:
1.  **Simulate Attacks:** Launch a Brute Force or Golden Ticket attack against the DC.
2.  **Detection Engineering:** Write a custom Wazuh Rule to detect the specific attack signature.
3.  **Response:** trigger an active response script to block the attacker IP.

---
*Lab maintained by [Your Name]*
