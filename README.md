# duplicacy Installation Guide

duplicacy is a free and open-source cloud backup tool. Duplicacy provides lock-free deduplication cloud backup tool

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
  - RAM: 512MB minimum
  - Storage: 10GB for cache
  - Network: Cloud storage APIs
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port N/A (default duplicacy port)
  - Web UI on 3875
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

# Install duplicacy
sudo dnf install -y duplicacy

# Enable and start service
sudo systemctl enable --now duplicacy

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
sudo firewall-cmd --reload

# Verify installation
duplicacy --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install duplicacy
sudo apt install -y duplicacy

# Enable and start service
sudo systemctl enable --now duplicacy

# Configure firewall
sudo ufw allow N/A

# Verify installation
duplicacy --version
```

### Arch Linux

```bash
# Install duplicacy
sudo pacman -S duplicacy

# Enable and start service
sudo systemctl enable --now duplicacy

# Verify installation
duplicacy --version
```

### Alpine Linux

```bash
# Install duplicacy
apk add --no-cache duplicacy

# Enable and start service
rc-update add duplicacy default
rc-service duplicacy start

# Verify installation
duplicacy --version
```

### openSUSE/SLES

```bash
# Install duplicacy
sudo zypper install -y duplicacy

# Enable and start service
sudo systemctl enable --now duplicacy

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
sudo firewall-cmd --reload

# Verify installation
duplicacy --version
```

### macOS

```bash
# Using Homebrew
brew install duplicacy

# Start service
brew services start duplicacy

# Verify installation
duplicacy --version
```

### FreeBSD

```bash
# Using pkg
pkg install duplicacy

# Enable in rc.conf
echo 'duplicacy_enable="YES"' >> /etc/rc.conf

# Start service
service duplicacy start

# Verify installation
duplicacy --version
```

### Windows

```bash
# Using Chocolatey
choco install duplicacy

# Or using Scoop
scoop install duplicacy

# Verify installation
duplicacy --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/duplicacy

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
duplicacy --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable duplicacy

# Start service
sudo systemctl start duplicacy

# Stop service
sudo systemctl stop duplicacy

# Restart service
sudo systemctl restart duplicacy

# Check status
sudo systemctl status duplicacy

# View logs
sudo journalctl -u duplicacy -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add duplicacy default

# Start service
rc-service duplicacy start

# Stop service
rc-service duplicacy stop

# Restart service
rc-service duplicacy restart

# Check status
rc-service duplicacy status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'duplicacy_enable="YES"' >> /etc/rc.conf

# Start service
service duplicacy start

# Stop service
service duplicacy stop

# Restart service
service duplicacy restart

# Check status
service duplicacy status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start duplicacy
brew services stop duplicacy
brew services restart duplicacy

# Check status
brew services list | grep duplicacy
```

### Windows Service Manager

```powershell
# Start service
net start duplicacy

# Stop service
net stop duplicacy

# Using PowerShell
Start-Service duplicacy
Stop-Service duplicacy
Restart-Service duplicacy

# Check status
Get-Service duplicacy
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream duplicacy_backend {
    server 127.0.0.1:N/A;
}

server {
    listen 80;
    server_name duplicacy.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name duplicacy.example.com;

    ssl_certificate /etc/ssl/certs/duplicacy.example.com.crt;
    ssl_certificate_key /etc/ssl/private/duplicacy.example.com.key;

    location / {
        proxy_pass http://duplicacy_backend;
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
    ServerName duplicacy.example.com
    Redirect permanent / https://duplicacy.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName duplicacy.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/duplicacy.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/duplicacy.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:N/A/
    ProxyPassReverse / http://127.0.0.1:N/A/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend duplicacy_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/duplicacy.pem
    redirect scheme https if !{ ssl_fc }
    default_backend duplicacy_backend

backend duplicacy_backend
    balance roundrobin
    server duplicacy1 127.0.0.1:N/A check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R duplicacy:duplicacy /etc/duplicacy
sudo chmod 750 /etc/duplicacy

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
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
sudo systemctl status duplicacy

# View logs
sudo journalctl -u duplicacy -f

# Monitor resource usage
top -p $(pgrep duplicacy)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/duplicacy"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/duplicacy-backup-$DATE.tar.gz" /etc/duplicacy /var/lib/duplicacy

echo "Backup completed: $BACKUP_DIR/duplicacy-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop duplicacy

# Restore from backup
tar -xzf /backup/duplicacy/duplicacy-backup-*.tar.gz -C /

# Start service
sudo systemctl start duplicacy
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u duplicacy -n 100
sudo tail -f /var/log/duplicacy/duplicacy.log

# Check configuration
duplicacy --version

# Check permissions
ls -la /etc/duplicacy
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep N/A

# Test connectivity
telnet localhost N/A

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep duplicacy)

# Check disk I/O
iotop -p $(pgrep duplicacy)

# Check connections
ss -an | grep N/A
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  duplicacy:
    image: duplicacy:latest
    ports:
      - "N/A:N/A"
    volumes:
      - ./config:/etc/duplicacy
      - ./data:/var/lib/duplicacy
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update duplicacy

# Debian/Ubuntu
sudo apt update && sudo apt upgrade duplicacy

# Arch Linux
sudo pacman -Syu duplicacy

# Alpine Linux
apk update && apk upgrade duplicacy

# openSUSE
sudo zypper update duplicacy

# FreeBSD
pkg update && pkg upgrade duplicacy

# Always backup before updates
tar -czf /backup/duplicacy-pre-update-$(date +%Y%m%d).tar.gz /etc/duplicacy

# Restart after updates
sudo systemctl restart duplicacy
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/duplicacy

# Clean old logs
find /var/log/duplicacy -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/duplicacy
```

## Additional Resources

- Official Documentation: https://docs.duplicacy.org/
- GitHub Repository: https://github.com/duplicacy/duplicacy
- Community Forum: https://forum.duplicacy.org/
- Best Practices Guide: https://docs.duplicacy.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
