## Home SOC Lab - Lessons Learned

A home security operations center built to develop hands on incident response and threat hunting skills. This lab simulated real-world attack techniques and detects them using enteprise-grade tooling.

## Key Lessons Learned

\## Session 1 — June 27, 2026



\### Splunk \& Sysmon Configuration

\- Splunk Universal Forwarder requires LocalSystem privileges to read Sysmon event logs

\- Windows TA must be installed on both the forwarder and the Splunk indexer for proper XML parsing

\- Raw .evtx file monitoring produces binary data — WinEventLog API inputs are required

\- Execution policy must be set to RemoteSigned for Atomic Red Team to run



\## Session 2 — June 27, 2026



\### Splunk Configuration Deep Dive

\- Splunk stanza is a config header in brackets — everything below applies to that log input

\- renderXml = true allows Splunk to parse XML and extract fields like EventCode and IpAddress

\- Splunk follows precedence order — etc\\system\\local takes priority over etc\\apps\\Splunk\_TA\_windows

\- The union command combines two separate searches into one dataset by normalizing a common field



\## Session 3 — June 29, 2026



\### Kali Linux on Hyper-V

\- Never use the Kali installer ISO on Hyper-V — graphical installer fails to render regardless of ISO version

\- Always use the pre-built Hyper-V VHDX from kali.org/get-kali → Virtual Machines tab

\- Run the included create-vm.ps1 script as Administrator — it sets up both network adapters automatically

\- Pre-built image requires hyperv-daemons installed for Enhanced Session to work

\- Package name in Kali 2026.1 is NOT hyperv-daemons — use: sudo apt install -y hyperv-daemons linux-image-cloud-amd64

\- Enable hv-kvp-daemon and hv-vss-daemon after install (hv-fcopy-daemon does not exist in this version)

\- Windows Firewall on ACCT-PC1/ACCT-PC2 blocks ICMP by default — add rule to allow ICMPv4 inbound

\- Shut down Kali from Hyper-V Manager, not from inside the VM, to avoid authentication prompt



\### Ubuntu Server / Wazuh Install

\- Ubuntu Server installer freezes on mirror check — hit Continue on "Mirror check failed" to skip it

\- Create swap file BEFORE running anything else — OOM killer will crash the VM without it

\- Swap commands: sudo fallocate -l 4G /swapfile \&\& sudo chmod 600 /swapfile \&\& sudo mkswap /swapfile \&\& sudo swapon /swapfile

\- Make swap persistent: echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

\- Wazuh install command: curl -o wazuh-install.sh https://packages.wazuh.com/4.7/wazuh-install.sh \&\& sudo bash wazuh-install.sh -a --ignore-check

\- Wazuh dashboard at https://192.168.10.30 — save admin password from install output immediately

## Author

Brandon Wilkinson — Cybersecurity Student at MSU Denver | Cybersecurity Intern at ULA  
[CyberBrief Automation Project](https://github.com/itzbwilk/cyberbrief)

