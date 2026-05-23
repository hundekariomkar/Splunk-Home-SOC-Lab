# 🔍 Splunk Home SOC Lab

A personal cybersecurity home lab for practicing SOC analyst skills using **Splunk Enterprise**, **Sysmon**, **Kali Linux**, and **Windows 10** inside Oracle VirtualBox.

---

## 📌 Lab Architecture

```
Home Network (192.168.19.x)
        |
Host Machine — Splunk Enterprise (192.168.19.100)
        |
   VirtualBox NAT Network (192.168.9.0/24)
   ┌─────────────────────────────┐
   │                             │
Kali Linux (192.168.9.1)  ←→  Windows 10 (192.168.9.2)
   Attacker                      Target / Victim
```

---

## 🖥️ Components

| Component | Role | IP |
|---|---|---|
| Host Machine (Windows) | Splunk Enterprise SIEM | 192.168.19.100 |
| Windows 10 VM | Target / Victim + Log Forwarder | 192.168.9.2 |
| Kali Linux VM | Attacker Machine | 192.168.9.1 |

---

## 🛠️ Tools Used

- **Splunk Enterprise** — SIEM for log collection and analysis
- **Splunk Universal Forwarder** — Sends logs from Windows 10 VM to Splunk
- **Sysmon (v15.20)** — Deep Windows event monitoring
- **Sysmon Config** — Olaf Hartong Sysmon Modular Config
- **Oracle VirtualBox** — Virtualization platform
- **Kali Linux** — Attack tools (nmap, metasploit, hydra)

---

## 📁 Repository Structure

```
splunk-lab/
│
├── README.md                  # This file
├── configs/
│   └── inputs.conf            # Splunk Universal Forwarder config
├── searches/
│   └── splunk_queries.md      # Useful SPL search queries
├── attack-scenarios/
│   └── scenarios.md           # Attack scenarios and detection steps
└── report/
    └── Splunk_Lab_Report.docx # Full lab setup report
```

---

## ⚙️ Setup Guide

### Step 1 — Splunk Enterprise on Host Machine
1. Download from [splunk.com](https://www.splunk.com)
2. Install on host machine
3. Enable receiving port 9997:
   - Settings → Forwarding and Receiving → Configure Receiving → New Port → `9997`
4. Add firewall rule:
```
netsh advfirewall firewall add rule name="Splunk9997" dir=in action=allow protocol=TCP localport=9997 profile=any enable=yes
```

---

### Step 2 — VirtualBox Network Setup
1. Create NAT Network in VirtualBox:
   - File → Tools → Network Manager → NAT Networks → Create
   - Name: `LabNetwork`, IPv4: `10.0.2.0/24`
2. Set both VMs to use `NAT Network → LabNetwork`

---

### Step 3 — Splunk Universal Forwarder on Windows 10 VM
1. Download from [splunk.com/universalforwarder](https://www.splunk.com/en_us/download/universal-forwarder.html)
2. Install with:
   - Username: `admin`
   - Receiving Indexer: `HOST-IP:9997`
3. Add forward server:
```
cd "C:\Program Files\SplunkUniversalForwarder\bin"
splunk add forward-server HOST-IP:9997 -auth admin:password
```

---

### Step 4 — Configure inputs.conf
Location: `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

```ini
[WinEventLog://Security]
index = main
disabled = 0

[WinEventLog://System]
index = main
disabled = 0

[WinEventLog://Application]
index = main
disabled = 0

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = main
disabled = 0

[WinEventLog://Microsoft-Windows-PowerShell/Operational]
index = main
disabled = 0
```

---

### Step 5 — Install Sysmon on Windows 10 VM
1. Download [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
2. Download config:
```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/olafhartong/sysmon-modular/master/sysmonconfig.xml" -OutFile "C:\Sysmon\sysmonconfig.xml"
```
3. Install:
```
cd C:\Sysmon
Sysmon64.exe -accepteula -i sysmonconfig.xml
```
4. Verify:
```
sc query Sysmon64
```

---

## 🔍 Key Splunk Searches

```spl
# See all log sources
index=main | stats count by sourcetype

# All event types
index=main | stats count by EventCode

# Successful logins
index=main EventCode=4624

# Failed login attempts
index=main EventCode=4625

# Network connections (Sysmon)
index=main EventCode=3 | stats count by SourceIp DestinationIp

# Process creation (Sysmon)
index=main EventCode=1 | table _time Image CommandLine

# All Sysmon events
index=main sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational"
| stats count by EventCode
```

---

## ⚔️ Attack Scenarios

| # | Attack | Kali Tool | Detection |
|---|---|---|---|
| 1 | Port Scan | `nmap -sS` | `EventCode=3` |
| 2 | Brute Force | `hydra` | `EventCode=4625` |
| 3 | Successful Exploit | Metasploit | `EventCode=4624` |
| 4 | Process Execution | Meterpreter | `EventCode=1` |
| 5 | Network Callback | Meterpreter | `EventCode=3` |

---

## 📊 Sysmon Event Codes Reference

| EventCode | Name | Detection Use |
|---|---|---|
| 1 | Process Created | Malicious execution |
| 3 | Network Connection | Port scans, C2 |
| 7 | Image Loaded | DLL injection |
| 11 | File Created | Malware dropping files |
| 12/13 | Registry Modified | Persistence |
| 22 | DNS Query | C2 beaconing |

---

## 🔒 Safety & Isolation

- ✅ Both VMs on isolated **NAT Network** — cannot reach home network
- ✅ Kali Linux **cannot reach** host machine or router
- ✅ All attacks **stay inside VirtualBox**
- ✅ VM **Snapshots** taken before each session
- ✅ No real malware — only built-in Kali tools

---

## 📈 Progress

- [x] Splunk Enterprise installed
- [x] Universal Forwarder configured
- [x] Windows Event Logs flowing
- [x] Sysmon installed and configured
- [x] 35,905+ events indexed
- [ ] Kali attack scenarios
- [ ] Splunk Dashboards
- [ ] Splunk Alerts

---

## 📚 Resources

- [Splunk Documentation](https://docs.splunk.com)
- [Sysmon by Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Olaf Hartong Sysmon Config](https://github.com/olafhartong/sysmon-modular)
- [MyDFIR YouTube Channel](https://www.youtube.com/@MyDFIR)
- [TryHackMe Splunk Rooms](https://tryhackme.com)

---

## ⚠️ Disclaimer

This lab is for **educational purposes only**. All attacks are performed in an isolated virtual environment. Do not use these techniques on systems you do not own or have permission to test.

---

*Made for SOC Analyst Practice*
