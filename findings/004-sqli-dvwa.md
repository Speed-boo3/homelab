# Finding 004 — SQL Injection (DVWA)

**Date:** 2026-04-29
**Target:** 192.168.56.30 (DVWA)
**Tool:** sqlmap, Burp Suite, manual testing
**MITRE ATT&CK:** T1190 — Exploit Public-Facing Application (Initial Access)
**Wazuh result:** ❌ Missed entirely

---

## What I did

Started with manual testing on the DVWA SQL injection page. Tried the classic single-quote test first.

Entered `1' OR '1'='1` in the user ID field. The page returned every user record in the database. That confirmed the injection point.

Then used sqlmap to automate the full extraction:

```bash
sqlmap -u "http://192.168.56.30/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=abc123;security=low" \
  --dump
```

sqlmap pulled the full users table — usernames and MD5 password hashes. Threw the hashes at hashcat with rockyou.txt and had plaintext passwords in under 10 seconds. MD5 with no salt in 2026 is essentially no protection.

---

## Why Wazuh missed it

This is the most interesting finding so far, because the miss is not a configuration issue — it is an architectural one.

Wazuh reads log files. The Apache access log recorded the HTTP request, including the query string with the injection payload. But Wazuh's default web rules are looking for status codes and basic patterns — they do not parse the actual content of query strings for injection syntax.

The request logged as a normal 200 OK. Nothing in the log looked unusual at a surface level.

This is why WAFs exist as a separate product. A WAF (ModSecurity, AWS WAF, Cloudflare) inspects HTTP payloads in transit before they reach the application. A SIEM is reading what happened after the fact. For web application attacks, you need both layers.

---

## Rule I added

A basic SQLi pattern match on Apache logs. It will catch obvious attacks but not encoded payloads — that is a limitation I will document.

```xml
<rule id="100010" level="10">
  <if_group>web_log</if_group>
  <regex>union.*select|'.*or.*'|1=1|--\s|xp_cmdshell</regex>
  <description>Possible SQL injection in web request — T1190</description>
  <mitre><id>T1190</id></mitre>
  <group>web_attack,sql_injection</group>
</rule>
```

---

## What would have stopped this

Parameterised queries are the only real fix. Everything else — input validation, WAF rules, encoding — can be bypassed with enough effort. A prepared statement cannot be injected because the query structure is fixed before user input is applied.

Beyond that: the passwords should never have been stored as unsalted MD5. bcrypt or Argon2 would have made the cracked hashes useless even after the database dump.

**Risk score:** Critical — full database access and credential extraction.
