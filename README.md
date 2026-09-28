
# wazuh-splunk-siem

<h2>Overview:</h2>

An advanced Security Information and Event Management (SIEM) and Extended Detection and Response (XDR) telemetry pipeline integrating Wazuh with Splunk Enterprise.This project establishes a centralized monitoring lab to capture, parse, and analyze endpoint security events from Linux target environments. 

Using Wazuh as the host-based EDR/XDR agent, system telemetry including authentication attempts, file integrity checks, and process audits is securely aggregated and forwarded to a Splunk backend.   

Within Splunk, raw JSON logs are normalized and transformed via custom SPL (Search Processing Language) logic to map security events against the MITRE ATT&CK framework. The resulting data powers a custom SOC operational dashboard designed using Splunk Dashboard Studio, providing real-time visibility into adversary tactics, multi-tier rule severity distributions, and actionable forensic drilldowns for incident response.

<h2>Architecture:</h2>

**Endpoint Layer (4-VM Lab):**

* **Windows 10:** Monitored using a full Wazuh agent combined with Sysmon for deep process, file, and network tracking.
* **Kali Linux & Ubuntu Server:** Monitored via standard Wazuh host-based agents tracking system logs, authentication, and file integrity.
* **Metasploitable 2:**  Integrated via agentless syslog forwarding to accommodate systems incapable of running modern agents.

**Parsing Layer:**

The Wazuh manager centralizes multi-source telemetry streams, parses raw data, and streams structured JSON alerts.

**Storage and Analytics layer (Splunk_enterprice)**

* Splunks ingest the JSON telemetry streams, 
* Custom Search Processing Language (SPL) normalizes fields (such as casting rule.level to numeric values) to track event velocities, rule severities, and threat mappings.

**Presentation Layer:**

Splunk Dashboard Studio renders the final operational interface, providing analysts with unified visibility into system health, attacker TTPs, and forensic details.
<img width="1470" height="930" alt="Screenshot 2026-09-27 at 3 25 03 AM" src="https://github.com/user-attachments/assets/8d3c57bf-70c2-4bcb-8807-74b77c70196e" />


<h2>Key Findings</h2>

1. **What the SIEM does**

Wazuh ingests security telemetry from three different endpoint types through three different collection methods, centralizes it into one dashboard, and applies rule-based detection with MITRE ATT&CK mapping, vulnerability scanning, and compliance benchmarking all running on a single Ubuntu host:
Windows 10: full agent + Sysmon, producing detailed process, network, and registry telemetry.
Kali Linux: full agent, standard host monitoring.
Metasploitable 2: agentless syslog forwarding, for a legacy 32-bit system that can't run a modern agent at all.


2. **Detection in action**

Repeated failed SSH logins against the Ubuntu host reliably triggered Wazuh's brute-force detection rule (rule 2502, MITRE T1110), and the Windows endpoint's Sysmon data was automatically mapped to real MITRE ATT&CK tactics (Persistence, Privilege Escalation, Defense Evasion) without any custom rule-writing all from Wazuh's default ruleset.


3. **A detection gap, found by testing**

Not every attack is visible to every monitoring method. Exploiting the vsftpd 2.3.4 backdoor (CVE-2011-2523) on Metasploitable  via both Metasploit and manually  gained root access that neither the syslog forwarding nor the agent-based monitoring detected, since the backdoor trigger operates at the network/protocol layer with no native logging. Closing this specific gap would require network-layer intrusion detection (e.g. Suricata) rather than log-based monitoring alone — a natural next addition to the lab.


<h2>Technical Challenges & Solutions: </h2>

* **Metasploitable Architecture Constraint:** Being 32-bit (i686), modern 64-bit Wazuh agents could not be installed. This was resolved by utilizing agentless syslog forwarding.
* **Sysmon Log Channel:** Sysmon data was not picked up by the Wazuh agent by default and required an explicit <localfile> configuration pointing to the Microsoft-Windows-Sysmon/Operational event channel.
* **Container Architecture & Permissions:** Native Splunk builds faced architecture and container permission hurdles, resolved via Docker emulation flags (--platform=linux/amd64) and correct host-side directory permissions (chmod -R 755).
* **VM Storage Constraints:** Internal disk space limitations caused by running indexers and multiple VMs were mitigated by migrating VM storage to an external SSD.

<h2>Tools Used</h2>

Wazuh · Splunk Enterprise · Kali Linux · Metasploitable 2 · Windows 10 · Ubuntu Server · Docker · Sysmon (SwiftOnSecurity configuration) · Metasploit

<h2>Lessons Learned:</h2>

Building this lab meant solving real integration problems, not just following a checklist architecture mismatches (32-bit targets, ARM64 hosts with no native Splunk build), configuration files that looked right but weren't being read by the actual running service, andpermission mismatches between a host and a container. Getting three different monitoring methods (agent, agent + Sysmon, syslog) all reporting into one SIEM, then layering a second analytics tool on top, gave a much clearer picture of how a real SOC's tooling actually fits together than any single-tool tutorial would have.

