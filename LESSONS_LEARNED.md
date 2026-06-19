## Home SOC Lab - Lessons Learned
A home security operations center built to develop hands on incident response and threat hunting skills. This lab simulated real-world attack techniques and detects them using enteprise-grade tooling.

## Key Lessons Learned

- I learned that Splunk Universal Forwarder requires LocalSystem privileges to read Sysmon event logs.
- I learned that Windows TA must be installed on both the forwarder and the Splunk indexer for proper XML parsing.
- I learned that raw .evtx file monitoring produces binary data — WinEventLog API inputs are required.
- I learned that execution policy must be set to RemoteSigned for Atomic Red Team to run.
- I learned that Splunk stanza is a way to config headers for log inputs. The header is in brackets and everything below that bracket are settings that apply to that header/log inputs.
- I learned that Splunk can easily parse information from XML format, but can't with plain-text. Using renderXml = true makes it easier to break the logs into the various fields. Without it, fields like EventCode and IpAddress wouldn't get extracted.
- I was running into an issue where I had the correct stanza in the wrong place. I learned that Splunk follows a precedence order on where it pulls the stanza from. The system level takes priority over the apps level. So, etc\system\local (location of incorrect stanza) has priority over etc\apps\Splunk_TA_windows (location of correct stanza).
- I learned how the union command combines two separate searches into one dataset. By normalizing a common field in each of the searches before union, I was able to find the relationship between the separate searches.

## Author

Brandon Wilkinson — Cybersecurity Student at MSU Denver | Cybersecurity Intern at ULA  
[CyberBrief Automation Project](https://github.com/itzbwilk/cyberbrief)
