# Lab Setup Guide

This is how I set up the lab. It is not the only way — but it works and the whole thing can be running in an afternoon.

---

## What you need

- A machine with at least 16GB RAM (the VMs need room to breathe)
- VirtualBox 7.0 — free from virtualbox.org
- About 50GB free disk space
- A few hours

---

## Download the VMs

| VM | Where to get it |
|---|---|
| Wazuh OVA | Ready-made appliance at packages.wazuh.com — saves a lot of time |
| Metasploitable 2 | sourceforge.net/projects/metasploitable — download the .zip |
| DVWA | github.com/digininja/DVWA — needs a LAMP stack, or use a pre-built VM |

---

## Network setup

Create a Host-Only network in VirtualBox: `192.168.56.0/24`. Assign all three VMs to it. This means the VMs can talk to each other but cannot reach the internet — important when you are running attacks.

---

## Install Wazuh 4.7

On a fresh Ubuntu 22.04 VM:

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

The install script handles everything — Wazuh manager, indexer and dashboard. Takes about 15 minutes. At the end it prints the admin password — save it somewhere.

Dashboard is at `https://192.168.56.10`. Accept the self-signed cert warning.

---

## Install Wazuh agents on targets

Do this on both Metasploitable and the DVWA VM:

```bash
# Debian/Ubuntu
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -
apt-get install wazuh-agent

# Set the manager IP
sed -i "s|MANAGER_IP|192.168.56.10|g" /var/ossec/etc/ossec.conf
systemctl start wazuh-agent
systemctl enable wazuh-agent
```

After a minute you should see both agents appear in the Wazuh dashboard under Agents.

---

## Add custom rules

Copy `configs/wazuh-rules/local_rules.xml` to:

```bash
/var/ossec/etc/rules/local_rules.xml
```

Then restart the manager:

```bash
systemctl restart wazuh-manager
```

---

## Verify everything is working

Run a quick SSH brute force from your host machine:

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.20 -t 2
```

Within 30 seconds you should see alerts appearing in the Wazuh dashboard. If you do, the pipeline is working.
