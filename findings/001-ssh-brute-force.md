# Finding 001 — SSH Brute Force

**Date:** 2026-04-28
**Target:** 192.168.56.20 (Metasploitable 2)
**Tool:** Hydra
**MITRE ATT&CK:** T1110 — Brute Force (Credential Access)
**Wazuh detected:** ✅ Yes

---

## What I did

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.20 -t 4
```

Hydra attempted 47 passwords before authenticating as `root` with password `toor` — the Metasploitable default.

---

## What Wazuh detected

Rule 5503 fired after the third failure. Rule 5715 (success after failures) fired when the login succeeded — the most important alert because it confirms the attacker got in.

---

## What Wazuh missed

Did not automatically block the IP. Wazuh Active Response can do this but requires configuration.

---

## Custom rule I added

```xml
<rule id="100001" level="12" frequency="5" timeframe="60">
  <if_matched_sid>5503</if_matched_sid>
  <same_source_ip/>
  <description>SSH brute force: $(frequency) failures from $(srcip) in 60s</description>
  <mitre><id>T1110</id></mitre>
</rule>
```

---

## Prevention

- `PermitRootLogin no` in sshd_config
- Key-based authentication only
- Wazuh Active Response to auto-block after N failures

**Severity:** High — root access obtained
