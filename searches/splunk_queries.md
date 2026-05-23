# Splunk Search Queries (SPL) Reference

## Basic Searches

```spl
# All events
index=main

# See all sourcetypes and count
index=main | stats count by sourcetype

# See all event codes and count
index=main | stats count by EventCode

# See all hosts sending logs
index=main | stats count by host
```

---

## Authentication Events

```spl
# Successful logins (EventCode 4624)
index=main EventCode=4624
| table _time host AccountName LogonType IpAddress

# Failed logins (EventCode 4625)
index=main EventCode=4625
| stats count by TargetUserName IpAddress
| sort -count

# Brute force detection (10+ failures)
index=main EventCode=4625
| stats count by TargetUserName IpAddress
| where count > 10
| sort -count
```

---

## Sysmon Events

```spl
# All Sysmon events by type
index=main sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational"
| stats count by EventCode

# Process creation (EventCode 1)
index=main EventCode=1
| table _time Image CommandLine ParentImage User

# Network connections (EventCode 3)
index=main EventCode=3
| table _time Image SourceIp SourcePort DestinationIp DestinationPort

# DNS queries (EventCode 22)
index=main EventCode=22
| table _time Image QueryName QueryResults

# File creation (EventCode 11)
index=main EventCode=11
| table _time Image TargetFilename

# Registry modifications (EventCode 13)
index=main EventCode=13
| table _time Image TargetObject Details
```

---

## Attack Detection

```spl
# Detect port scanning (many connections from one IP)
index=main EventCode=3
| stats count by SourceIp DestinationPort
| where count > 50
| sort -count

# Detect nmap scan
index=main EventCode=3
| stats dc(DestinationPort) as ports_scanned by SourceIp
| where ports_scanned > 20

# Detect suspicious processes
index=main EventCode=1
| search Image="*cmd.exe*" OR Image="*powershell.exe*" OR Image="*mshta.exe*"
| table _time Image CommandLine ParentImage

# Detect lateral movement
index=main EventCode=4624 LogonType=3
| stats count by AccountName IpAddress
| sort -count
```

---

## Internal / Health Checks

```spl
# Verify forwarder is connected
index=_internal source=*metrics.log* group=tcpin_connections

# Check Splunk internal errors
index=_internal sourcetype=splunkd component=TcpInputProc

# Check indexing rate
index=_internal source=*metrics.log* group=per_index_thruput series=main
| timechart sum(kb) as KB_indexed
```
