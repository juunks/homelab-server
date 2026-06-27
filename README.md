# homelab-server
Personal home server built on Ubuntu Server 24.04 LTS
# Homelab Server

A personal home server built on a spare laptop running Ubuntu Server 24.04 LTS. Designed as a systems administration portfolio project, separate from university lab work.

## Hardware
- Laptop: Dell (repurposed spare)
- OS: Ubuntu Server 24.04 LTS
- Storage: 500GB HDD
- Network: Ethernet (static IP 192.168.1.6)

## Services

| Service | Purpose | Port |
|---|---|---|
| BIND9 | Local DNS server (homelab.local) | 53 |
| Apache2 + SSL | HTTPS web server with self-signed certificate | 80, 443 |
| Samba | Network file sharing for Windows clients | 445 |
| Fail2ban | SSH brute force protection | - |
| WireGuard | VPN for encrypted remote access | 51820 |
| UFW | Firewall — only required ports open | - |

## Network Architecture
- Server IP: 192.168.1.6 (static)
- Local domain: homelab.local
- VPN subnet: 10.0.0.0/24
- DNS forwarders: 8.8.8.8, 8.8.4.4

## Security
- UFW firewall with minimal open ports
- Self-signed SSL certificate for HTTPS
- Fail2ban bans IPs after 5 failed SSH attempts within 10 minutes
- SSH password authentication disabled — key-based auth only
- WireGuard VPN using ed25519 keys

## Repository Structure
- configs/ — all service configuration files  
- screenshots/ — proof of working services
## What I Learned

This project was built entirely on real hardware, which introduced problems that don't exist in virtual machine labs:

- **Network troubleshooting** — the ethernet interface showed NO-CARRIER repeatedly due to a loose cable. Diagnosed using `dmesg | grep enp3s0` which showed the link flapping up and down.
- **Netplan configuration** — Ubuntu Server 24.04 uses Netplan instead of traditional ifconfig. Had to manually configure DHCP and then static IP via YAML.
- **WireGuard permissions** — PostUp UFW rules failed because the wg-quick service doesn't have permission to run UFW. Solved by switching to direct iptables rules.
- **SSH hardening** — disabled password authentication only after confirming key-based login worked, to avoid locking myself out.

## Installation Overview

**UFW**
```bash
sudo ufw allow OpenSSH
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 53
sudo ufw allow 51820/udp
sudo ufw allow samba
sudo ufw enable
```

**BIND9**
```bash
sudo apt install bind9 bind9utils -y
# Configure /etc/bind/named.conf.options (forwarders, listen-on, allow-query)
# Configure /etc/bind/named.conf.local (zone declaration)
# Create /etc/bind/zones/db.homelab.local (A records)
sudo systemctl restart bind9
```

**Apache2 + SSL**
```bash
sudo apt install apache2 -y
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/homelab.key \
  -out /etc/ssl/certs/homelab.crt
sudo a2enmod ssl
sudo a2ensite homelab.conf
sudo a2dissite 000-default.conf
sudo systemctl restart apache2
```

**Samba**
```bash
sudo apt install samba -y
sudo mkdir -p /srv/samba/shared
sudo chmod 777 /srv/samba/shared
# Add share config to /etc/samba/smb.conf
sudo systemctl restart smbd
```

**Fail2ban**
```bash
sudo apt install fail2ban -y
# Create /etc/fail2ban/jail.local with sshd jail config
sudo systemctl restart fail2ban
```

**WireGuard**
```bash
sudo apt install wireguard -y
wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
# Create /etc/wireguard/wg0.conf with Interface and Peer sections
sudo systemctl enable wg-quick@wg0
sudo wg-quick up wg0
```

**SSH Hardening**
```bash
# Generate ed25519 key pair on client machine
ssh-keygen -t ed25519
# Copy public key to server authorized_keys
# Set PasswordAuthentication no in /etc/ssh/sshd_config
sudo systemctl restart sshd
```
