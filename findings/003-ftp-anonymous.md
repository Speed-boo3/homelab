# Finding 003 — FTP Anonymous Login

**Date:** 2026-04-29
**Target:** 192.168.56.20 (Metasploitable 2)
**Tool:** ftp client
**MITRE ATT&CK:** T1078 — Valid Accounts (Initial Access)
**Wazuh detected:** ✅ Yes — Rule 11204

---

## What I did

```bash
ftp 192.168.56.20
Name: anonymous
Password: [blank]
# Connected — file listing accessible
```

---

## What Wazuh detected

Rule 11204 fired immediately: "Anonymous FTP login attempt."

---

## Prevention

In vsftpd.conf: `anonymous_enable=NO`
Replace FTP with SFTP entirely.

**Severity:** Medium — information disclosure, escalates with vsftpd backdoor
