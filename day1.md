Day 1: Building the "Mini SOC" – Architecture & Troubleshooting
Date: December 17, 2025 Role: Junior SOC Analyst / Security Engineer Objective: Establish network segmentation, verify telemetry, and stabilize the SIEM (Wazuh).

1. The Environment Setup
I designed a segmented home lab to mimic a corporate network, isolating "Attackers," "Victims," and "Management" into separate VLANs using pfSense.

Firewall/Router: pfSense (Gateway)

Attacker: Kali Linux (10.0.1.x - VLAN 10)

SIEM: Wazuh Server (10.0.2.x - VLAN 20)

Victims: Windows 11 & Metasploitable (10.0.3.x & 10.0.4.x)

Hypervisor: VMware Workstation

2. The Challenges (Troubleshooting Log)
Day 1 was dedicated to infrastructure debugging. Connectivity is the foundation of security; you cannot defend a network you cannot reach.

Issue 1: The "Silent" Network (Inter-VLAN Routing)
Symptom: My Kali machine (Attacker) could not communicate with the Wazuh Server. Ping requests resulted in 100% packet loss.

Diagnosis: I verified the IP routing tables (ip route) and confirmed "Default Gateways" were set correctly. The issue was traced to VMware Virtual Switches. The VMs were physically plugged into the wrong "Virtual Cables" (VMnets), meaning traffic never reached the pfSense router.

The Fix:

Mapped pfSense interfaces (em1, em2) to specific VMware Custom Networks (VMnet2, VMnet3).

Aligned Kali and Wazuh Network Adapters to match these specific VMnets.

Verified connectivity by pinging the Gateway (10.0.1.1).

Issue 2: The Firewall Block
Symptom: Even after fixing the cables, traffic between subnets was dropped.

Diagnosis: pfSense acts as a "Deny All" firewall by default between new interfaces.

The Fix:

Temporary: Used the shell command pfctl -d to disable packet filtering and confirm visibility.

Permanent: Created a "Pass Any" firewall rule on the LAN interface to allow the Attacker subnet to talk to the SIEM subnet.

Issue 3: The "Frozen" Dashboard (Resource Exhaustion)
Symptom: The Wazuh Dashboard would timeout ("Connection Lost") or spin indefinitely, even when the network was working.

Diagnosis: The Wazuh Indexer (Database) and OS were consuming 95% of the 4GB RAM assigned to the VM, leaving no resources for the Web UI.

The Lesson: Enterprise tools like Elastic/OpenSearch are resource-hungry.

Solution In-Progress: Increasing VM RAM to 6GB+ or implementing a Swap File to prevent OOM (Out of Memory) crashes.

Issue 4: Layer 3 vs. Layer 7 Connectivity
Symptom: I could ping the gateway (Layer 3 success), but the browser would time out loading the page (Layer 7 failure).

Diagnosis: A classic "Hacker" mistake—my browser still had the Burp Suite Proxy configuration active, trying to route traffic to a tool that wasn't running.

The Fix: Disabled the system proxy in Firefox to restore web access.

3. Automation Implemented
To keep the lab lightweight and sustainable, I began scripting a cleanup policy:

Cron Job: Scheduled a task to run every hour (0 * * * *) to delete raw log archives older than 6 hours.

ISM Policy: Configured Wazuh Index Management to delete database indices after 6 hours, creating a "Flash Lab" that resets daily.

4. Key Takeaway
Today proved that Network Segmentation is difficult to configure but essential for security. I learned that having a "Ping" doesn't mean you have a "Service," and that resource management (RAM) is just as critical as firewall rules in a SOC environment.

Next Shift Goals:

Stabilize Wazuh RAM usage.

Perform the first "Live Fire" Brute Force attack.

Verify alert generation in the dashboard.
