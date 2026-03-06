# 🔒 Deploy & Secure an Azure Linux VM

> **Hands-on Azure security hardening lab — layered protection on an Ubuntu VM using NSG, UFW, fail2ban, and SSH key authentication**

![Azure](https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Ubuntu_24.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![SSH](https://img.shields.io/badge/SSH-Key_Auth-4A90D9?style=for-the-badge&logo=openssh&logoColor=white)
![Security](https://img.shields.io/badge/fail2ban-Enabled-green?style=for-the-badge)

---

## 📋 Overview

Deployed an Ubuntu 24.04 Linux VM on Microsoft Azure and applied a layered security hardening approach — restricting access via Network Security Groups, disabling password authentication in favor of SSH key pairs, configuring a host-based firewall, installing brute force protection, and enabling automatic security updates.

**Every control addresses a specific attack vector. Defense in depth.**

---

## 🎯 Objectives

- Deploy an Ubuntu VM on Azure using SSH key pair authentication
- Restrict SSH access to a single trusted IP via NSG rule
- Configure UFW host-based firewall as a second layer of network defense
- Disable password authentication to eliminate brute force attack surface
- Install fail2ban to auto-ban repeated failed connection attempts
- Enable automatic security updates for ongoing vulnerability patching

---

## 🏗️ Lab Environment

| Component | Details |
|-----------|---------|
| Platform | Microsoft Azure (Free Subscription) |
| VM Name | vm-001 |
| OS | Ubuntu Server 24.04 LTS |
| Auth Method | SSH key pair (.pem) — no password login |
| Access Control | NSG restricted to personal IP only |
| Firewall | UFW + fail2ban |

---

## 🛡️ Security Controls Applied

| Control | Threat Mitigated |
|---------|-----------------|
| ✅ NSG IP Restriction | Blocks all SSH access except from your IP — eliminates external scanning |
| ✅ SSH Key Authentication | Removes the password attack surface — keys are cryptographically unguessable |
| ✅ UFW Firewall | Host-based layer blocking unexpected traffic even if NSG rules change |
| ✅ Password Auth Disabled | No password means no brute force vector — even if port 22 is reached |
| ✅ fail2ban | Auto-bans IPs after repeated failed attempts — stops enumeration and scanning |
| ✅ Automatic Updates | Patches CVEs without manual intervention — closes known vulnerabilities continuously |

---

## ⚙️ Setup Steps

### 1. Create a Resource Group
```
Azure Portal → Resource Groups → + Create
Name: az-vm-ssh | Region: your nearest region
```
> 💡 Keep all resources in the same region to avoid connectivity issues.

---

### 2. Deploy the Ubuntu VM
```
Virtual Machines → Create → Azure Virtual Machine
Name: vm-001 | Image: Ubuntu Server 24.04 LTS
Size: Standard_B1s | Authentication: SSH public key
```
- Set **Authentication type** to SSH public key
- Select **Generate new key pair** — Azure will create a `.pem` file
- Download and store the `.pem` file securely — **you cannot reconnect without it**

---

### 3. Restrict NSG to Your IP Only
```
vm-001-nsg → Inbound security rules → Edit SSH rule
Source: IP Addresses
Source IP: [your personal IP — check whatismyip.com]
Destination port: 22 | Protocol: TCP | Action: Allow | Priority: 100
```
> 🔒 Only your IP can now reach port 22. All other connections are silently dropped.

---

### 4. Connect via SSH
```bash
ssh -i "path/to/vm-001_key.pem" azureuser@<VM_PUBLIC_IP>
```
- Find your VM's public IP on the Azure Portal Overview page
- Accept the host fingerprint on first connection (`yes`)

---

### 5. Update the OS
```bash
sudo apt update && sudo apt upgrade -y
```
> Always patch immediately after deploying. This closes known vulnerabilities before anything else is configured.

---

### 6. Configure UFW Firewall
```bash
sudo apt install ufw -y
sudo ufw allow ssh          # ⚠️ Do this BEFORE enabling or you'll lock yourself out
sudo ufw enable
sudo ufw status             # Verify — should show SSH ALLOW
```

Expected output:
```
Status: active
To          Action    From
--          ------    ----
22/tcp      ALLOW     Anywhere
22/tcp (v6) ALLOW     Anywhere (v6)
```

---

### 7. Disable Password Authentication
```bash
sudo nano /etc/ssh/sshd_config
```

Find and set these three values:
```
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM yes
```

Save (`Ctrl+X → Y → Enter`) then restart SSH:
```bash
sudo systemctl restart sshd
```
> ✅ Password logins are now disabled. Only your .pem key can authenticate.

---

### 8. Install fail2ban
```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban --now
sudo systemctl status fail2ban      # Verify: should show active (running)
```

Default behavior:
- Monitors `/var/log/auth.log` for failed SSH attempts
- Bans offending IPs for **10 minutes** after **5 failed attempts**
- Customize thresholds in `/etc/fail2ban/jail.local`

---

### 9. Enable Automatic Security Updates
```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
# Select Yes in the dialog
```
> 🔄 Security patches now install automatically — critical CVEs get patched without waiting for manual maintenance.

---

## 📋 Quick Command Reference

| Command | Purpose |
|---------|---------|
| `sudo apt update && sudo apt upgrade -y` | Update OS and all packages |
| `sudo apt install ufw -y` | Install UFW firewall |
| `sudo ufw allow ssh && sudo ufw enable` | Allow SSH and activate firewall |
| `sudo ufw status` | Check firewall rules |
| `sudo nano /etc/ssh/sshd_config` | Edit SSH daemon config |
| `sudo systemctl restart sshd` | Reload SSH config |
| `sudo apt install fail2ban -y` | Install brute force protection |
| `sudo systemctl enable fail2ban --now` | Enable and start fail2ban |
| `sudo systemctl status fail2ban` | Verify fail2ban is running |

---

## ✅ Outcomes

- [x] Ubuntu VM deployed on Azure with SSH key pair — no password auth
- [x] NSG rule restricts SSH to single trusted IP address
- [x] UFW firewall active with only port 22 allowed
- [x] Password authentication disabled in sshd_config
- [x] fail2ban installed and monitoring for brute force attempts
- [x] Automatic security updates enabled

---

## 🧰 Tools & Technologies

- **Microsoft Azure** — Cloud infrastructure and NSG configuration
- **Ubuntu 24.04 LTS** — Linux server OS
- **OpenSSH** — Key-based authentication
- **UFW** — Host-based firewall (Uncomplicated Firewall)
- **fail2ban** — Intrusion prevention / brute force protection
- **unattended-upgrades** — Automatic security patching

---

## 📚 What I Learned

- How to deploy and access a Linux VM on Azure using SSH key authentication
- Difference between network-layer security (NSG) vs host-layer security (UFW) — why both matter
- How to harden SSH configuration to eliminate password-based attack vectors
- How fail2ban monitors auth logs and dynamically bans malicious IPs
- Why automatic updates are a critical part of any security baseline

---

## 🔗 Related Projects

- [Azure Honeypot + Microsoft Sentinel SIEM Lab](../azure-honeypot-sentinel) — Live attack detection and geolocation mapping

---

*Built 2025 — Azure Linux VM Security Hardening Lab*
