# gollum Installation Guide

gollum is a free and open-source git-backed wiki. Gollum provides simple wiki system backed by Git repository

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
  - CPU: 1 core minimum
  - RAM: 256MB minimum
  - Storage: 500MB for data
  - Network: HTTP access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 4567 (default gollum port)
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

# Install gollum
sudo dnf install -y gollum

# Enable and start service
sudo systemctl enable --now gollum

# Configure firewall
sudo firewall-cmd --permanent --add-port=4567/tcp
sudo firewall-cmd --reload

# Verify installation
gollum --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install gollum
sudo apt install -y gollum

# Enable and start service
sudo systemctl enable --now gollum

# Configure firewall
sudo ufw allow 4567

# Verify installation
gollum --version
```

### Arch Linux

```bash
# Install gollum
sudo pacman -S gollum

# Enable and start service
sudo systemctl enable --now gollum

# Verify installation
gollum --version
```

### Alpine Linux

```bash
# Install gollum
apk add --no-cache gollum

# Enable and start service
rc-update add gollum default
rc-service gollum start

# Verify installation
gollum --version
```

### openSUSE/SLES

```bash
# Install gollum
sudo zypper install -y gollum

# Enable and start service
sudo systemctl enable --now gollum

# Configure firewall
sudo firewall-cmd --permanent --add-port=4567/tcp
sudo firewall-cmd --reload

# Verify installation
gollum --version
```

### macOS

```bash
# Using Homebrew
brew install gollum

# Start service
brew services start gollum

# Verify installation
gollum --version
```

### FreeBSD

```bash
# Using pkg
pkg install gollum

# Enable in rc.conf
echo 'gollum_enable="YES"' >> /etc/rc.conf

# Start service
service gollum start

# Verify installation
gollum --version
```

### Windows

```bash
# Using Chocolatey
choco install gollum

# Or using Scoop
scoop install gollum

# Verify installation
gollum --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/gollum

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
gollum --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable gollum

# Start service
sudo systemctl start gollum

# Stop service
sudo systemctl stop gollum

# Restart service
sudo systemctl restart gollum

# Check status
sudo systemctl status gollum

# View logs
sudo journalctl -u gollum -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add gollum default

# Start service
rc-service gollum start

# Stop service
rc-service gollum stop

# Restart service
rc-service gollum restart

# Check status
rc-service gollum status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'gollum_enable="YES"' >> /etc/rc.conf

# Start service
service gollum start

# Stop service
service gollum stop

# Restart service
service gollum restart

# Check status
service gollum status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start gollum
brew services stop gollum
brew services restart gollum

# Check status
brew services list | grep gollum
```

### Windows Service Manager

```powershell
# Start service
net start gollum

# Stop service
net stop gollum

# Using PowerShell
Start-Service gollum
Stop-Service gollum
Restart-Service gollum

# Check status
Get-Service gollum
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream gollum_backend {
    server 127.0.0.1:4567;
}

server {
    listen 80;
    server_name gollum.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name gollum.example.com;

    ssl_certificate /etc/ssl/certs/gollum.example.com.crt;
    ssl_certificate_key /etc/ssl/private/gollum.example.com.key;

    location / {
        proxy_pass http://gollum_backend;
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
    ServerName gollum.example.com
    Redirect permanent / https://gollum.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName gollum.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/gollum.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/gollum.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:4567/
    ProxyPassReverse / http://127.0.0.1:4567/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend gollum_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/gollum.pem
    redirect scheme https if !{ ssl_fc }
    default_backend gollum_backend

backend gollum_backend
    balance roundrobin
    server gollum1 127.0.0.1:4567 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R gollum:gollum /etc/gollum
sudo chmod 750 /etc/gollum

# Configure firewall
sudo firewall-cmd --permanent --add-port=4567/tcp
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
sudo systemctl status gollum

# View logs
sudo journalctl -u gollum -f

# Monitor resource usage
top -p $(pgrep gollum)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/gollum"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/gollum-backup-$DATE.tar.gz" /etc/gollum /var/lib/gollum

echo "Backup completed: $BACKUP_DIR/gollum-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop gollum

# Restore from backup
tar -xzf /backup/gollum/gollum-backup-*.tar.gz -C /

# Start service
sudo systemctl start gollum
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u gollum -n 100
sudo tail -f /var/log/gollum/gollum.log

# Check configuration
gollum --version

# Check permissions
ls -la /etc/gollum
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 4567

# Test connectivity
telnet localhost 4567

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep gollum)

# Check disk I/O
iotop -p $(pgrep gollum)

# Check connections
ss -an | grep 4567
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  gollum:
    image: gollum:latest
    ports:
      - "4567:4567"
    volumes:
      - ./config:/etc/gollum
      - ./data:/var/lib/gollum
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update gollum

# Debian/Ubuntu
sudo apt update && sudo apt upgrade gollum

# Arch Linux
sudo pacman -Syu gollum

# Alpine Linux
apk update && apk upgrade gollum

# openSUSE
sudo zypper update gollum

# FreeBSD
pkg update && pkg upgrade gollum

# Always backup before updates
tar -czf /backup/gollum-pre-update-$(date +%Y%m%d).tar.gz /etc/gollum

# Restart after updates
sudo systemctl restart gollum
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/gollum

# Clean old logs
find /var/log/gollum -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/gollum
```

## Additional Resources

- Official Documentation: https://docs.gollum.org/
- GitHub Repository: https://github.com/gollum/gollum
- Community Forum: https://forum.gollum.org/
- Best Practices Guide: https://docs.gollum.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
