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
