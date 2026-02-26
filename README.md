# SOC Lab 1 – Windows Sysmon Log Ingestion with Splunk

## Overview

This lab demonstrates end-to-end Windows log ingestion into Splunk using Sysmon and the Splunk Universal Forwarder.

The objective was to configure, troubleshoot, and validate a working SOC-style log pipeline suitable for monitoring and detection engineering.

---

## Lab Environment

- Windows 11 VM
- Sysmon (Microsoft System Monitor)
- Splunk Universal Forwarder
- Ubuntu VM running Dockerized Splunk Enterprise
- TCP Port 9997 for log forwarding

![Docker Container Running](screenshots/01-docker-container-running.png)
![Docker Container Healthy](screenshots/03-docker-container-healthy.png)
![Splunk Login Page](screenshots/04-splunk-login-page.png)
![Splunk Home Dashboard](screenshots/05-splunk-home-dashboard.png)

---

## Architecture

Windows 11  
→ Sysmon generates logs  
→ Splunk Universal Forwarder  
→ TCP 9997  
→ Splunk Enterprise (Ubuntu Docker)  
→ Indexed into:
- wineventlog
- sysmon

This architecture simulates a simplified SOC telemetry pipeline where endpoint telemetry is centrally aggregated and analyzed within a SIEM platform.
---

## Index Creation

Two custom indexes were created in Splunk:

- `wineventlog`
- `sysmon`

Verified in:

Settings → Indexes

### Custom Windows Index Created

![Custom Windows Index Created](screenshots/08-custom-windows-index-created.png)

## Receiving Port Configuration

Splunk was configured to receive logs on TCP port 9997.

![Receiving Port 9997 Enabled](screenshots/splunk-receiving-port-9997.png)
---

## Universal Forwarder Configuration

![Forwarder Installed](screenshots/forwarder-already-installed.png)

![Forwarder Service Running](screenshots/forwarder-service-running.png)

![Forwarder Restart Success](screenshots/10-forwarder-restart-success.png)
---

### inputs.conf
```ini
[WinEventLog://Application]
index = wineventlog

[WinEventLog://Security]
index = wineventlog

[WinEventLog://System]
index = wineventlog

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
renderXml = true
index = sysmon
```
### outputs.conf
```ini
[tcpout]
defaultGroup = indexer_group

[tcpout:indexer_group]
server = <Splunk_Server_IP>:9997
useACK = true
```
The SplunkForwarder service was configured to run as:

LocalSystem

This was required to allow access to the Sysmon event channel.

## Verification – Windows Event Logs

SPL Used:
```
index=wineventlog | stats count by sourcetype
```
![Wineventlog Verification](screenshots/06-wineventlog-ingestion-verification.png)

Confirmed ingestion of:
- Application logs
- Security logs
- System logs
- Firewall logs

---

## Verification – Sysmon Ingestion

SPL Used:

```
index=sysmon
```

![Sysmon Ingestion Success](screenshots/05_sysmon_ingestion_success.png)

![Windows VM Sysmon Event Viewer](screenshots/02_windows_vm_event_viewer_sysmon.png)

Confirmed:
- Event ID 1 (Process Creation)
- CommandLine field visibility
- ParentImage
- Hash values
- Sourcetype: XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
  
---  

## Issues Encountered & Resolution

![No Data Ingested Yet](screenshots/07_No_Data_Ingested_Yet.png)

![Sysmon Wide Search No Results](screenshots/Sysmon_WideSearch_NoResults.png)

![Command Syntax Error](screenshots/Troubleshooting_03_Command_Syntax_Error.png)

![Guest Additions Installed](screenshots/Troubleshooting_04_GuestAdditions_Installed.png)

---

### Issue: Sysmon logs not ingesting
Symptoms:
- wineventlog index populated
- sysmon index empty

Root Cause:
The SplunkForwarder service was running under the default service account:
NT SERVICE\SplunkForwarder.

This account does not have sufficient privileges to access the
Microsoft-Windows-Sysmon/Operational event channel.

Resolution:
Changed service account to:
LocalSystem

Restarted forwarder service.

Result:
Sysmon logs successfully ingested into the sysmon index.

## Skills Demonstrated

- Splunk index configuration
- Universal Forwarder deployment
- Windows Event Log monitoring
- Sysmon configuration
- Service account troubleshooting
- btool validation
- SPL verification
- End-to-end ingestion validation
- SOC-style log pipeline debugging

---

## Snapshot Strategy

A final stable VM snapshot was created to preserve the validated ingestion state:

Lab1_Log_Ingestion_Sysmon_Working_Final

This allows rollback to a verified working ingestion state.

---

## Outcome

Successfully built, configured, and validated a Windows log ingestion pipeline suitable for SOC monitoring and detection workflows.

---
