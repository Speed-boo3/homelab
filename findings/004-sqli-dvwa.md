# Finding 004 — SQL Injection (DVWA)

**Date:** 2026-04-29
**Target:** 192.168.56.30 (DVWA)
**Tool:** sqlmap, manual
**MITRE ATT&CK:** T1190 — Exploit Public-Facing Application (Initial Access)
**Wazuh detected:** ❌ Missed

---

## What I did

Manual test: input `1' OR '1'='1` returned all user records.

```bash
sqlmap -u "http://192.168.56.30/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=abc123;security=low" --dump
```

Extracted full users table including MD5 password hashes. Cracked with hashcat in under 10 seconds.

---

## Why Wazuh missed it

Wazuh reads logs — it does not inspect HTTP payloads. Apache logged the request but default rules do not parse query strings for injection patterns.

---

## Custom rule I added

```xml
<rule id="100010" level="10">
  <if_group>web_log</if_group>
  <regex>union.*select|'.*or.*'|1=1|--\s</regex>
  <description>Possible SQL injection attempt — T1190</description>
  <mitre><id>T1190</id></mitre>
</rule>
```

---

## Prevention

- Parameterised queries — eliminates SQLi completely
- WAF (ModSecurity, AWS WAF)
- Least-privilege database user

**Severity:** Critical — full database dump including credentials
