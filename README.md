<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0d1117,40:0a0f1a,100:0a1a0a&height=200&section=header&text=Home%20Security%20Lab&fontSize=52&fontColor=00ff41&animation=fadeIn&fontAlignY=42&desc=Wazuh%20SIEM%20%7C%20Simulated%20Attacks%20%7C%20MITRE%20ATT%26CK%20Findings%20%7C%20Incident%20Reports&descAlignY=65&descColor=446644&descSize=13"/>

<img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&size=14&duration=2500&pause=900&color=00FF41&center=true&vCenter=true&width=680&lines=Wazuh+SIEM+on+Ubuntu+VM;Simulated+attacks+against+Metasploitable+2+and+DVWA;MITRE+ATT%26CK-tagged+detection+findings;Real+rules+built+from+real+attacks"/>

<br/>

![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-0d1117?style=flat-square&logoColor=00ff41)
![Platform](https://img.shields.io/badge/Platform-VirtualBox-0d1117?style=flat-square&logoColor=00ff41)
![OS](https://img.shields.io/badge/OS-Ubuntu%2022.04-0d1117?style=flat-square&logo=ubuntu&logoColor=ff9900)
![Status](https://img.shields.io/badge/Status-Active-0d1117?style=flat-square&logoColor=00ff41)

</div>

---

## What this is

A personal security lab built to practise real attack and detection techniques. Every finding in this repo came from running an actual attack against a vulnerable VM, watching what Wazuh detected — and what it missed.

The point is not just to run tools. The point is to understand the full loop: attack happens, log is generated, alert fires, analyst investigates, rule is improved. That loop is what real SOC work looks like.

---

## Lab setup

```
┌─────────────────────────────────────────────┐
│  VirtualBox — Host machine                  │
│                                             │
│  ┌──────────────────┐  ┌─────────────────┐  │
│  │  Ubuntu 22.04    │  │ Metasploitable 2│  │
│  │  Wazuh SIEM      │  │ DVWA            │  │
│  │  (Manager +      │  │ (Attack targets)│  │
│  │   Dashboard)     │  │                 │  │
│  └──────────────────┘  └─────────────────┘  │
│                                             │
│  Internal network: 192.168.56.0/24          │
└─────────────────────────────────────────────┘
```

**VMs:**

| VM | OS | Role | IP |
|---|---|---|---|
| wazuh-server | Ubuntu 22.04 | SIEM manager + dashboard | 192.168.56.10 |
| target-01 | Metasploitable 2 | Vulnerable Linux target | 192.168.56.20 |
| target-02 | Ubuntu + DVWA | Vulnerable web app | 192.168.56.30 |

---

## Findings

Each finding documents one attack, what Wazuh detected, what it missed and what rule I added.

| # | Attack | MITRE technique | Wazuh detected | Finding report |
|---|---|---|---|---|
| 001 | SSH brute force | T1110 — Brute Force | ✅ Yes | [findings/001-ssh-brute-force.md](findings/001-ssh-brute-force.md) |
| 002 | Nmap port scan | T1046 — Network Service Discovery | ⚠️ Partial | [findings/002-nmap-scan.md](findings/002-nmap-scan.md) |
| 003 | FTP anonymous login | T1078 — Valid Accounts | ✅ Yes | [findings/003-ftp-anonymous.md](findings/003-ftp-anonymous.md) |
| 004 | SQL injection (DVWA) | T1190 — Exploit Public App | ❌ Missed | [findings/004-sqli-dvwa.md](findings/004-sqli-dvwa.md) |
| 005 | Directory traversal | T1083 — File Discovery | ⚠️ Partial | [findings/005-directory-traversal.md](findings/005-directory-traversal.md) |

*More findings added as the lab grows.*

---

## How I run each test

1. Start all VMs, confirm Wazuh dashboard is live
2. Run the attack from Kali or the host machine
3. Check what Wazuh generated — alerts, log entries, rule IDs
4. Document exactly what fired and what did not
5. Write or improve a detection rule to cover any gaps
6. Add the finding to this repo

---

## Detection rules I have written

Custom Wazuh rules built from lab findings. Stored in `configs/wazuh-rules/`.

```xml
<!-- Rule: SSH brute force — 5+ failed logins in 60 seconds -->
<rule id="100001" level="10" frequency="5" timeframe="60">
  <if_matched_sid>5503</if_matched_sid>
  <same_source_ip/>
  <description>SSH brute force: $(frequency) failed logins from $(srcip)</description>
  <mitre>
    <id>T1110</id>
  </mitre>
  <group>authentication_failures,brute_force</group>
</rule>

<!-- Rule: FTP anonymous login -->
<rule id="100002" level="8">
  <if_sid>11200</if_sid>
  <match>anonymous</match>
  <description>FTP anonymous login — T1078 Valid Accounts</description>
  <mitre>
    <id>T1078</id>
  </mitre>
</rule>
```

---

## Tools used

| Tool | Purpose |
|---|---|
| Wazuh | SIEM — log collection, alerting, dashboards |
| VirtualBox | Virtualisation |
| Metasploitable 2 | Intentionally vulnerable Linux target |
| DVWA | Vulnerable PHP web application |
| Nmap | Port scanning and service enumeration |
| Hydra | Brute force testing |
| Nikto | Web server vulnerability scanning |
| SQLmap | SQL injection testing |
| Wireshark | Packet capture and analysis |

---

## Lab setup guide

Full setup instructions are in [configs/setup-guide.md](configs/setup-guide.md).

Short version:

```bash
# 1. Install Wazuh on Ubuntu 22.04
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash ./wazuh-install.sh -a

# 2. Access the dashboard
# https://192.168.56.10 (default creds in setup guide)

# 3. Add an agent on the target VM
# Follow configs/agent-setup.md
```

---

## Folder structure

```
homelab/
├── findings/
│   ├── 001-ssh-brute-force.md
│   ├── 002-nmap-scan.md
│   ├── 003-ftp-anonymous.md
│   ├── 004-sqli-dvwa.md
│   └── 005-directory-traversal.md
├── configs/
│   ├── setup-guide.md
│   ├── agent-setup.md
│   └── wazuh-rules/
│       └── local_rules.xml
├── playbooks/
│   └── incident-response-template.md
└── scripts/
    └── generate-finding.sh
```

---

## Related projects

This lab feeds directly into my other projects:

- Detection rules tested here are refined into [soc-project](https://github.com/Speed-boo3/soc-project) detection engineering
- Risk findings go into [grc-project](https://github.com/Speed-boo3/grc-project) risk register
- Cloud misconfigurations tested here map to [cloud-security](https://github.com/Speed-boo3/cloud-security) scanners

<div align="center">
<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0a1a0a,50:0a0f1a,100:0d1117&height=100&section=footer&text=Attack.%20Detect.%20Improve.&fontSize=15&fontColor=00ff41&animation=twinkling"/>
</div>
