# Lab Setup Guide

## Requirements

- Host machine with 16GB+ RAM
- VirtualBox (free) or VMware Workstation
- ~50GB free disk space

---

## Step 1 — Download the VMs

| VM | Download | Notes |
|---|---|---|
| Wazuh OVA | [packages.wazuh.com](https://packages.wazuh.com) | Ready-made OVA available |
| Metasploitable 2 | [sourceforge.net/projects/metasploitable](https://sourceforge.net/projects/metasploitable) | Intentionally vulnerable Linux |
| DVWA | [github.com/digininja/DVWA](https://github.com/digininja/DVWA) | Vulnerable PHP web app |

---

## Step 2 — Network setup in VirtualBox

Create a Host-Only network: `192.168.56.0/24`

Assign each VM to this network so they can communicate but are isolated from the internet.

---

## Step 3 — Install Wazuh

On Ubuntu 22.04:

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

Access the dashboard at `https://192.168.56.10`
Default credentials are shown at end of install — change them immediately.

---

## Step 4 — Install Wazuh agent on targets

On each target VM:

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -
apt-get install wazuh-agent
# Set manager IP to 192.168.56.10
systemctl start wazuh-agent
```

---

## Step 5 — Add custom rules

Copy `configs/wazuh-rules/local_rules.xml` to:
```
/var/ossec/etc/rules/local_rules.xml
```

Restart Wazuh: `systemctl restart wazuh-manager`
