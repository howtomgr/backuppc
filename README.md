# backuppc Installation Guide

backuppc is a free and open-source PC backup system. BackupPC provides high-performance enterprise backup system

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 100GB for backups
  - Network: SMB/rsync/tar
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default backuppc port)
  - None
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install backuppc
sudo dnf install -y backuppc

# Enable and start service
sudo systemctl enable --now backuppc

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
backuppc --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install backuppc
sudo apt install -y backuppc

# Enable and start service
sudo systemctl enable --now backuppc

# Configure firewall
sudo ufw allow 80

# Verify installation
backuppc --version
```

### Arch Linux

```bash
# Install backuppc
sudo pacman -S backuppc

# Enable and start service
sudo systemctl enable --now backuppc

# Verify installation
backuppc --version
```

### Alpine Linux

```bash
# Install backuppc
apk add --no-cache backuppc

# Enable and start service
rc-update add backuppc default
rc-service backuppc start

# Verify installation
backuppc --version
```

### openSUSE/SLES

```bash
# Install backuppc
sudo zypper install -y backuppc

# Enable and start service
sudo systemctl enable --now backuppc

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
backuppc --version
```

### macOS

```bash
# Using Homebrew
brew install backuppc

# Start service
brew services start backuppc

# Verify installation
backuppc --version
```

### FreeBSD

```bash
# Using pkg
pkg install backuppc

# Enable in rc.conf
echo 'backuppc_enable="YES"' >> /etc/rc.conf

# Start service
service backuppc start

# Verify installation
backuppc --version
```

### Windows

```bash
# Using Chocolatey
choco install backuppc

# Or using Scoop
scoop install backuppc

# Verify installation
backuppc --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/backuppc

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
backuppc --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable backuppc

# Start service
sudo systemctl start backuppc

# Stop service
sudo systemctl stop backuppc

# Restart service
sudo systemctl restart backuppc

# Check status
sudo systemctl status backuppc

# View logs
sudo journalctl -u backuppc -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add backuppc default

# Start service
rc-service backuppc start

# Stop service
rc-service backuppc stop

# Restart service
rc-service backuppc restart

# Check status
rc-service backuppc status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'backuppc_enable="YES"' >> /etc/rc.conf

# Start service
service backuppc start

# Stop service
service backuppc stop

# Restart service
service backuppc restart

# Check status
service backuppc status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start backuppc
brew services stop backuppc
brew services restart backuppc

# Check status
brew services list | grep backuppc
```

### Windows Service Manager

```powershell
# Start service
net start backuppc

# Stop service
net stop backuppc

# Using PowerShell
Start-Service backuppc
Stop-Service backuppc
Restart-Service backuppc

# Check status
Get-Service backuppc
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream backuppc_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name backuppc.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name backuppc.example.com;

    ssl_certificate /etc/ssl/certs/backuppc.example.com.crt;
    ssl_certificate_key /etc/ssl/private/backuppc.example.com.key;

    location / {
        proxy_pass http://backuppc_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName backuppc.example.com
    Redirect permanent / https://backuppc.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName backuppc.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/backuppc.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/backuppc.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend backuppc_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/backuppc.pem
    redirect scheme https if !{ ssl_fc }
    default_backend backuppc_backend

backend backuppc_backend
    balance roundrobin
    server backuppc1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R backuppc:backuppc /etc/backuppc
sudo chmod 750 /etc/backuppc

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status backuppc

# View logs
sudo journalctl -u backuppc -f

# Monitor resource usage
top -p $(pgrep backuppc)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/backuppc"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/backuppc-backup-$DATE.tar.gz" /etc/backuppc /var/lib/backuppc

echo "Backup completed: $BACKUP_DIR/backuppc-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop backuppc

# Restore from backup
tar -xzf /backup/backuppc/backuppc-backup-*.tar.gz -C /

# Start service
sudo systemctl start backuppc
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u backuppc -n 100
sudo tail -f /var/log/backuppc/backuppc.log

# Check configuration
backuppc --version

# Check permissions
ls -la /etc/backuppc
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep backuppc)

# Check disk I/O
iotop -p $(pgrep backuppc)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  backuppc:
    image: backuppc:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/backuppc
      - ./data:/var/lib/backuppc
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update backuppc

# Debian/Ubuntu
sudo apt update && sudo apt upgrade backuppc

# Arch Linux
sudo pacman -Syu backuppc

# Alpine Linux
apk update && apk upgrade backuppc

# openSUSE
sudo zypper update backuppc

# FreeBSD
pkg update && pkg upgrade backuppc

# Always backup before updates
tar -czf /backup/backuppc-pre-update-$(date +%Y%m%d).tar.gz /etc/backuppc

# Restart after updates
sudo systemctl restart backuppc
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/backuppc

# Clean old logs
find /var/log/backuppc -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/backuppc
```

## Additional Resources

- Official Documentation: https://docs.backuppc.org/
- GitHub Repository: https://github.com/backuppc/backuppc
- Community Forum: https://forum.backuppc.org/
- Best Practices Guide: https://docs.backuppc.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
