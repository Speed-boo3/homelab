# Finding 002 — Nmap Port Scan

**Date:** 2026-04-28
**Target:** 192.168.56.20 (Metasploitable 2)
**Tool:** Nmap
**MITRE ATT&CK:** T1046 — Network Service Discovery (Discovery)
**Wazuh detected:** ⚠️ Partial

---

## What I did

```bash
nmap -sS -sV -O 192.168.56.20
```

---

## Key findings

| Port | Service | Risk |
|---|---|---|
| 21 | vsftpd 2.3.4 | **Critical** — contains a backdoor (CVE-2011-2523) |
| 23 | Telnet | High — credentials sent in plaintext |
| 3306 | MySQL 5.0.51a | High — database exposed to network |
| 5900 | VNC | High — remote desktop, no encryption |

vsftpd 2.3.4 has a backdoor triggered by sending `:)` in the username — opens a shell on port 6200.

---

## What Wazuh missed

No dedicated port scan detection. Wazuh needs Suricata integration for network-level IDS.

---

## Prevention

- Firewall: whitelist management IPs only
- Remove vsftpd 2.3.4 immediately
- Disable Telnet — replace with SSH
- Block MySQL port 3306 at the firewall

**Severity:** High — multiple critical vulnerabilities exposed
