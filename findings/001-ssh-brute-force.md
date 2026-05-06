# Finding 001 — SSH Brute Force

**Date:** 2026-04-28
**Target:** 192.168.56.20 (Metasploitable 2)
**Tool:** Hydra
**MITRE ATT&CK:** T1110 — Brute Force (Credential Access)
**Wazuh result:** ✅ Detected

---

## What I did

Started simple. SSH brute force is one of the most common attacks in the wild and a reasonable first test to see whether Wazuh was working properly.

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.20 -t 4
```

Hydra tried 47 passwords before it hit `toor` — the Metasploitable default root password. The whole thing took about 90 seconds.

---

## What Wazuh caught

Rule 5503 fired after the third failed login, which was faster than I expected. What was more interesting was Rule 5715 — SSH authentication success after multiple failures. That is the alert that actually matters, because it means the attacker got in, not just that they tried.

If I were running a real SOC and saw 5503 followed by 5715 from the same IP, that is an escalation. The brute force worked.

---

## What it missed

Wazuh detected the attack but did nothing about it. The IP was not blocked. In a production environment this is where Active Response comes in — Wazuh can call a firewall rule automatically when a rule fires. I have not set that up yet, which is a gap I want to close.

---

## Rule I added

The default threshold for 5503 is quite loose. I tightened it — 5 failures in 60 seconds feels more appropriate for catching a burst attack without generating noise from a user who just forgot their password.

```xml
<rule id="100001" level="12" frequency="5" timeframe="60">
  <if_matched_sid>5503</if_matched_sid>
  <same_source_ip/>
  <description>SSH brute force: $(frequency) failures from $(srcip) in 60 seconds</description>
  <mitre><id>T1110</id></mitre>
  <group>brute_force,authentication_failures</group>
</rule>
```

---

## What would have stopped this

The obvious fix is disabling password authentication entirely and requiring SSH keys. `PermitRootLogin no` in sshd_config would have also prevented the root login specifically. Wazuh Active Response could have blocked the IP after 5 failures, which would have made the attack impractical even with a valid wordlist.

**Risk score:** Likelihood 5, Impact 4, Score 20 — Critical. Root access was obtained.
