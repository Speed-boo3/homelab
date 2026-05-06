# Finding 002 — Nmap Port Scan

**Date:** 2026-04-28
**Target:** 192.168.56.20 (Metasploitable 2)
**Tool:** Nmap 7.94
**MITRE ATT&CK:** T1046 — Network Service Discovery (Discovery)
**Wazuh result:** ⚠️ Partial — saw connection spikes, no dedicated scan alert

---

## What I did

Before attacking anything properly I wanted to know what was running. Standard SYN scan with service and OS detection.

```bash
nmap -sS -sV -O 192.168.56.20
```

The results were honestly alarming — this is a VM that is supposed to be vulnerable, but seeing it laid out like this makes the risk concrete.

---

## What was found

| Port | Service | The problem |
|---|---|---|
| 21 | vsftpd 2.3.4 | Has a backdoor hardcoded into the source — see below |
| 23 | Telnet | Everything travels in plaintext, including passwords |
| 3306 | MySQL 5.0.51a | A database port should never be reachable from outside |
| 5900 | VNC | Remote desktop with no encryption |

The vsftpd 2.3.4 issue is worth calling out. Someone injected a backdoor into the source code in 2011 — sending `:)` in the username field opens a root shell on port 6200. It is 15 years old and still shows up on vulnerable VMs because it perfectly illustrates what a supply chain attack looks like in practice. This is exactly the kind of thing NIS2 and ISO 27001:2022 Annex A 5.19 are concerned about.

---

## What Wazuh caught

Not much. The auth log showed increased connection noise but no specific scan alert fired. This is a known limitation — Wazuh is built around log analysis, not network packet inspection. To catch scans properly you need Suricata running alongside it. That is on my list.

---

## What would have stopped this

A properly configured firewall would have made most of these ports invisible. The scan would still have run but it would have found almost nothing. Beyond that — vsftpd 2.3.4 should have been replaced, Telnet should have been removed entirely, and MySQL should only be reachable from the application server.

**Risk score:** High — exposed services reveal the full attack surface.
