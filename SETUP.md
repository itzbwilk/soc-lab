## Home SOC Lab Setup
A home security operations center built to develop hands on incident response and threat hunting skills. This lab simulated real-world attack techniques and detects them using enteprise-grade tooling.

## Lab Architecture
- **Hypervisor** Microsoft Hyper-V on Windows 11
- **Target VM** Windows 10 Pro (Desktop-8Q984MR)
- **Attack VM** Kali Linux
- **SIEM** Splunk Enterprise (free tier) on host machine
- **Endpoint Logging** Sysmon v15 with SwiftOnSecurity config
- **Log Forwarding** Splunk Universal Forwarder -> Splunk Enterprise via TCP 9997
- **Attack Simulation** Atomic Red Team (Invoke-AtomicTest)

## Setup

### Prerequisites
- Windows 10/11 host with Hyper-V enabled
- Splunk Enterprise (free tier) installed on host
- Windows 10 VM running in Hyper-V
- Kali Linux VM running in Hyper-V (attack simulation)

### Installation Steps

1. Install Sysmon on target VM with SwiftOnSecurity config:
```
.\Sysmon64.exe -accepteula -i sysmonconfig-export.xml
```

2. Install Splunk Universal Forwarder on target VM, point to host IP on port 9997

3. Install Splunk Add-on for Microsoft Windows on Splunk host

4. Open port 9997 on host firewall:
```
New-NetFirewallRule -DisplayName "Splunk Forwarder" -Direction Inbound -Protocol TCP -LocalPort 9997 -Action Allow
```

5. Configure inputs.conf on forwarder (`etc\system\local\inputs.conf`):
```
[WinEventLog://Security]
index = main
sourcetype = XmlWinEventLog:Security
disabled = 0
renderXml = true

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = main
sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
disabled = 0
renderXml = true
```
**Important:** Place this inputs.conf in `etc\system\local\` on the forwarder, not in the Splunk_TA_windows app folder. Splunk's config precedence gives `etc\system\local` priority over app-level configs — misplacing this stanza was a major troubleshooting issue during lab setup.

6. Enable Windows Firewall logging on target VM:
```
Set-NetFirewallProfile -Profile Domain,Public,Private -LogAllowed True -LogBlocked True -LogFileName "%systemroot%\system32\LogFiles\Firewall\pfirewall.log"
```

7. Add firewall log monitor to inputs.conf:
```
[monitor://C:\Windows\System32\LogFiles\Firewall\pfirewall.log]
index = main
sourcetype = WindowsFirewallLog
disabled = false
```

8. Install Atomic Red Team on target VM:
```
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
Install-AtomicRedTeam -getAtomics -Force
```

9. Enable RDP on target VM:
```
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayName "Remote Desktop*"
```

## Author

Brandon Wilkinson — Cybersecurity Student at MSU Denver | Cybersecurity Intern at ULA  
[CyberBrief Automation Project](https://github.com/itzbwilk/cyberbrief)
