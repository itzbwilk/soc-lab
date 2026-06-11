## Home SOC Lab - Splunk + Sysmon + Atomic Red Team
A home security operations center built to develop hands on incident response and threat hunting skills. This lab simulated real-world attack techniques and detects them using enteprise-grade tooling.

## Lab Architecture
- **Hypervisor** Microsoft Hyper-V on Windows 11
- **Target VM** Windows 10 Pro (Desktop-8Q984MR)
- **SIEM** Splunk Enterprise (free tier) on host machine
- **Endpoint Logging** Sysmon v15 with SwiftOnSecurity config
- **Log Forwarding** Splunk Universal Forwarder -> Splunk Enterprise via TCP 9997
- **Attack Simulation** Atomic Red Team (Invoke-AtomicTest)

## Log Sources
| Source | Sourcetype | Description |
|--------|-----------|-------------|
| Windows Security Log | WinEventLog | Authentication events, failed logins |
| Sysmon Operational | XmlWinEventLog | Process creation, network connections, file events |

## Detections Built
| Detection | MITRE Technique | SPL Search |
|-----------|----------------|------------|
| Failed Login Brute Force | T1110 | EventCode=4625 |
| PowerShell Execution | T1059.001 | EventCode=1 CommandLine=*powershell* |
| Credential Dumping | T1003.001 | EventCode=1 Image=*dumpert* |
| System Discovery | T1082 | EventCode=1 |

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

- Splunk Universal Forwarder requires LocalSystem privileges to read Sysmon event logs
- Windows TA must be installed on both the forwarder and the Splunk indexer for proper XML parsing
- Raw .evtx file monitoring produces binary data — WinEventLog API inputs are required
- Execution policy must be set to RemoteSigned for Atomic Red Team to run

## Next Steps

- [ ] Add Kali Linux VM for network-based attack simulation
- [ ] Build automated alerting for brute force detection
- [ ] Simulate lateral movement with T1021.001
- [ ] Build correlation searches for multi-stage attack detection
- [ ] Document full incident report from simulated attack chain

## Author

Brandon Wilkinson — Cybersecurity Student at MSU Denver | Cybersecurity Intern at ULA  
[CyberBrief Automation Project](https://github.com/itzbwilk/cyberbrief)










   
