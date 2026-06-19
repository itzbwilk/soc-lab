## Home SOC Lab - Splunk + Sysmon + Atomic Red Team
A home security operations center built to develop hands on incident response and threat hunting skills. This lab simulated real-world attack techniques and detects them using enteprise-grade tooling.

## Lab Architecture
- **Hypervisor** Microsoft Hyper-V on Windows 11
- **Target VM** Windows 10 Pro (Desktop-8Q984MR)
- **Attack VM** Kali Linux
- **SIEM** Splunk Enterprise (free tier) on host machine
- **Endpoint Logging** Sysmon v15 with SwiftOnSecurity config
- **Log Forwarding** Splunk Universal Forwarder -> Splunk Enterprise via TCP 9997
- **Attack Simulation** Atomic Red Team (Invoke-AtomicTest)

## Log Sources
| Source | Sourcetype | Description |
|--------|-----------|-------------|
| Windows Security Log | WinEventLog | Authentication events, failed logins |
| Sysmon Operational | XmlWinEventLog | Process creation, network connections, file events |
| Windows Firewall Log | WindowsFirewallLog | Inbound/outbound connection records

## Detections Built
| Detection | MITRE Technique | SPL Search |
|-----------|----------------|------------|
| Inbound Port Scan Detection | T1046 | WindowsFirewallLog direction=RECEIVE count > 10 |
| Failed Login Brute Force | T1110 | EventCode=4625 |
| PowerShell Execution | T1059.001 | EventCode=1 CommandLine=*powershell* |
| Credential Dumping | T1003.001 | EventCode=1 Image=*dumpert* |
| System Discovery | T1082 | EventCode=1 |
| Multi-Stage Attack Correlation | T1046 + T1110.001 | Correlated port scans and brute force events from src_ip across WindowsFirewallLog and XmlWinEventLog |

## Correlation Searches

### Port Scan -> Brute Force Correlation
Detects a single source IP that appears in two seperate sourcetypes within a 60 minute window. Indicates a multi-stage attack where an attacker had performed reconnaissance before attempting access.

index=main sourcetype="WindowsFirewallLog" earliest=-60m latest=now
| eval attacker_ip=src_ip
| union [search index=main sourcetype="XmlWinEventLog" earliest=-60m latest=now | eval attacker_ip=IpAddress]
| stats dc(sourcetype) as source_count count by attacker_ip
| where source_count > 1

## Attack Simulations Run

### T1059.001 — PowerShell Execution
Simulated PowerShell-based execution using Atomic Red Team.
Detected via Sysmon EventCode 1 (Process Creation) in Splunk.

### T1003.001 — Credential Dumping (LSASS)
Simulated LSASS memory access using Outflank Dumpert technique.
Detected via Sysmon process creation events showing dumpert execution and whoami 
reconnaissance activity post-dump.

### T1082 — System Discovery
Simulated attacker reconnaissance including Griffon recon framework and Machine GUID 
enumeration. Detected via process creation logs showing discovery tool execution.

### T1046 — Network Port Scan
Ran nmap from Kali Linux against Windows VM. Detected via Windows Firewall log monitoring inbound RECEIVE connections exceeding threshold from single source IP.

### T1110.001 — RDP Brute Force
Ran Hydra from Kali Linux against RDP (Port 3389) using rockyou.txt wordlist. Detected via Security log EventCode 4625 failed login events exceeding threshold from single source IP. 

## Dashboard

Built a SOC Lab Dashboard in Splunk with three panels:
- Failed Logins — tracks authentication failures by user and host
- Process Creation — monitors all process spawning with full command lines
- Network Connections — tracks outbound connections by process and destination

## Setup

### Prerequisites
- Windows 10/11 host with Hyper-V enabled
- Splunk Enterprise (free tier) installed on host
- Windows 10 VM running in Hyper-V

### Installation Steps

1. Install Sysmon on target VM with SwiftOnSecurity config:
  .\Sysmon64.exe -accepteula -i sysmonconfig-export.xml
2. Install Splunk Universal Forwarder on target VM, point to host IP on port 9997
3. Configure inputs.conf on forwarder:
  [WinEventLog://Security]
  index = main
  disabled = 0
  [WinEventLog://Microsoft-Windows-Sysmon/Operational]
  index = main
  disabled = 0
  renderXml = true
4. Install Splunk Add-on for Microsoft Windows on Splunk host
5. Open port 9997 on host firewall:
  New-NetFirewallRule -DisplayName "Splunk Forwarder" -Direction Inbound -Protocol TCP -LocalPort 9997 -Action Allow
6. Install Atomic Red Team on target VM:
  IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
  Install-AtomicRedTeam -getAtomics -Force

## Key Lessons Learned

- Splunk Universal Forwarder requires LocalSystem privileges to read Sysmon event logs.
- Windows TA must be installed on both the forwarder and the Splunk indexer for proper XML parsing.
- Raw .evtx file monitoring produces binary data — WinEventLog API inputs are required.
- Execution policy must be set to RemoteSigned for Atomic Red Team to run.
- Splunk stanza is a way to config headers for log inputs. The header is in brackets and everything below that bracket are settings that apply to that header/log inputs.
- I learned that Splunk can easily parse information from XML format, but can't with plain-text. Using renderXml = true makes it easier to break the logs into the various fields. Without it, fields like EventCode and IpAddress wouldn't get extracted.
- I was running into an issue where I had the correct stanza in the wrong place. I learned that Splunk follows a precedence order on where it pulls the stanza from. The system level takes priority over the apps level. So, etc\system\local (location of incorrect stanza) has priority over etc\apps\Splunk_TA_windows (location of correct stanza).

## Next Steps

- [ ] Simulate lateral movement with T1021.001
- [ ] Build correlation searches for multi-stage attack detection
- [ ] Document full incident report from simulated attack chain

## Author

Brandon Wilkinson — Cybersecurity Student at MSU Denver | Cybersecurity Intern at ULA  
[CyberBrief Automation Project](https://github.com/itzbwilk/cyberbrief)










   
