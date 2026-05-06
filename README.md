<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0d1117,40:0a0f1a,100:0a1a0a&height=200&section=header&text=Home%20Security%20Lab&fontSize=52&fontColor=00ff41&animation=fadeIn&fontAlignY=42&desc=Wazuh%20SIEM%20%7C%20Simulated%20Attacks%20%7C%20MITRE%20ATT%26CK%20Findings&descAlignY=65&descColor=446644&descSize=13"/>

<img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&size=14&duration=2500&pause=900&color=00FF41&center=true&vCenter=true&width=680&lines=Wazuh+SIEM+4.7+on+Ubuntu+22.04;Metasploitable+2+and+DVWA+as+targets;MITRE+ATT%26CK-tagged+findings;Custom+detection+rules+written+from+scratch"/>

<br/>

![Wazuh](https://img.shields.io/badge/SIEM-Wazuh%204.7-0d1117?style=flat-square&logoColor=00ff41)
![Platform](https://img.shields.io/badge/Platform-VirtualBox-0d1117?style=flat-square&logoColor=00ff41)
![OS](https://img.shields.io/badge/Ubuntu-22.04-0d1117?style=flat-square&logo=ubuntu&logoColor=ff9900)
![Status](https://img.shields.io/badge/Status-Active-0d1117?style=flat-square&logoColor=00ff41)

</div>

---

## Why I built this

I kept reading about SIEM tools in textbooks and writing about them in assignments. At some point that starts to feel hollow — you can describe how Wazuh works without actually knowing what it feels like to watch an alert fire in real time, or to discover that your rules missed something obvious.

So I built a lab. Two vulnerable VMs, Wazuh running on Ubuntu, and a simple rule: every time I run an attack, I write it up. What fired, what didn't, and what I changed because of it. This repo is that documentation.

Most of the findings are not surprising — SSH brute force gets caught, SQL injection does not unless you configure it properly. But going through the process of understanding *why* something is missed, and then writing a rule to fix it, is the kind of thing you can't get from reading.

---

## Lab environment

Three VMs on a VirtualBox host-only network (`192.168.56.0/24`), isolated from the internet.

```
┌──────────────────────────────────────────────────┐
│  VirtualBox host                                 │
│                                                  │
│  ┌────────────────────┐   ┌────────────────────┐ │
│  │  Ubuntu 22.04      │   │  Metasploitable 2  │ │
│  │  Wazuh 4.7         │   │  + DVWA            │ │
│  │  192.168.56.10     │   │  192.168.56.20/30  │ │
│  └────────────────────┘   └────────────────────┘ │
│                                                  │
│  Attacker: host machine or Kali VM               │
└──────────────────────────────────────────────────┘
```

| VM | Role | IP |
|---|---|---|
| Ubuntu 22.04 + Wazuh 4.7 | SIEM manager and dashboard | 192.168.56.10 |
| Metasploitable 2 | Intentionally vulnerable Linux target | 192.168.56.20 |
| Ubuntu 22.04 + DVWA | Vulnerable web application | 192.168.56.30 |

Wazuh agents are installed on both targets so logs flow to the manager automatically.

---

## Findings

Every finding follows the same structure: what I did, what Wazuh caught, what it missed, and what rule I added or changed. The missed detections are often more interesting than the ones that worked.

| # | Attack | MITRE technique | Wazuh | Report |
|---|---|---|---|---|
| 001 | SSH brute force | T1110 — Brute Force | ✅ Detected | [findings/001-ssh-brute-force.md](findings/001-ssh-brute-force.md) |
| 002 | Nmap port scan | T1046 — Network Service Discovery | ⚠️ Partial | [findings/002-nmap-scan.md](findings/002-nmap-scan.md) |
| 003 | FTP anonymous login | T1078 — Valid Accounts | ✅ Detected | [findings/003-ftp-anonymous.md](findings/003-ftp-anonymous.md) |
| 004 | SQL injection (DVWA) | T1190 — Exploit Public App | ❌ Missed | [findings/004-sqli-dvwa.md](findings/004-sqli-dvwa.md) |
| 005 | Directory traversal | T1083 — File Discovery | ⚠️ Partial | [findings/005-directory-traversal.md](findings/005-directory-traversal.md) |

*Adding more as the lab develops.*

---

## Custom detection rules

The rules I have written based on what the lab missed. All stored in [`configs/wazuh-rules/local_rules.xml`](configs/wazuh-rules/local_rules.xml).

Finding 004 (SQL injection) was the most useful — Wazuh's default rules are log-based and completely miss HTTP payload attacks. Writing a rule for it made me realise why WAFs exist as a separate layer rather than being built into a SIEM.

```xml
<!-- SSH brute force — tighter window than Wazuh default -->
<rule id="100001" level="12" frequency="5" timeframe="60">
  <if_matched_sid>5503</if_matched_sid>
  <same_source_ip/>
  <description>SSH brute force: $(frequency) failures from $(srcip) in 60s</description>
  <mitre><id>T1110</id></mitre>
</rule>

<!-- SQL injection patterns in Apache logs — catches what default rules miss -->
<rule id="100010" level="10">
  <if_group>web_log</if_group>
  <regex>union.*select|'.*or.*'|1=1|--\s</regex>
  <description>Possible SQL injection in web request — T1190</description>
  <mitre><id>T1190</id></mitre>
</rule>
```

---

## Tools

| Tool | What I use it for |
|---|---|
| Wazuh 4.7 | SIEM — log collection, correlation, custom rules, dashboards |
| VirtualBox 7.0 | Virtualisation |
| Metasploitable 2 | Vulnerable Linux target — intentionally broken |
| DVWA | Vulnerable PHP web app — good for web attack testing |
| Hydra | Brute force — SSH, FTP, HTTP forms |
| Nmap 7.94 | Port scanning, service fingerprinting, OS detection |
| SQLmap | SQL injection testing and exploitation |
| Nikto | Web server misconfiguration scanning |
| Burp Suite Community | Manual HTTP request inspection and manipulation |
| Wireshark | Packet capture to verify what attacks actually look like on the wire |

---

## What I have learned so far

**Wazuh is strong on log-based detection and weak on network/payload-level attacks.** It catches SSH brute force reliably because auth logs are detailed. It misses SQL injection completely unless you write custom rules that parse Apache log query strings — and even then, encoding bypasses are easy.

**The gap between "alert fired" and "incident understood" is larger than I expected.** Wazuh told me a brute force was happening. It did not tell me whether the attacker succeeded. Correlating the failure alerts with the eventual successful login required me to write a second rule. That correlation step is most of what a real L1 analyst does.

**Writing rules teaches you more than reading them.** Every rule I have written started with something being missed. That failure-first process is much better for understanding detection logic than copying rules from a ruleset.

---

## What is next

- Set up Suricata as a network IDS and feed it into Wazuh — that should close the gap on port scans and payload-level attacks
- Test privilege escalation techniques (T1548) and document what Wazuh sees in kernel and sudo logs
- Add a Windows VM target and test credential dumping (T1003) — most enterprise environments are Windows-heavy and it is worth understanding how detection differs from Linux

---

## Setup

Full instructions: [`configs/setup-guide.md`](configs/setup-guide.md)

Short version:
```bash
# Install Wazuh 4.7 on Ubuntu 22.04
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
# Dashboard at https://192.168.56.10
```

---

## Related projects

What I find in this lab feeds into my other work:

- Detection rules tested here are refined into [soc-project](https://github.com/Speed-boo3/soc-project)
- Vulnerabilities found become risk register entries in [grc-project](https://github.com/Speed-boo3/grc-project)
- Cloud misconfigurations map to [cloud-security](https://github.com/Speed-boo3/cloud-security)

<div align="center">
<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0a1a0a,50:0a0f1a,100:0d1117&height=100&section=footer&text=Attack.%20Detect.%20Improve.&fontSize=15&fontColor=00ff41&animation=twinkling"/>
</div>
