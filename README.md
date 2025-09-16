# librenms Installation Guide

librenms is a free and open-source network monitoring. LibreNMS provides autodiscovering network monitoring platform

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
  - Storage: 10GB for data
  - Network: SNMP/HTTP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default librenms port)
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

# Install librenms
sudo dnf install -y librenms

# Enable and start service
sudo systemctl enable --now librenms

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
librenms --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install librenms
sudo apt install -y librenms

# Enable and start service
sudo systemctl enable --now librenms

# Configure firewall
sudo ufw allow 80

# Verify installation
librenms --version
```

### Arch Linux

```bash
# Install librenms
sudo pacman -S librenms

# Enable and start service
sudo systemctl enable --now librenms

# Verify installation
librenms --version
```

### Alpine Linux

```bash
# Install librenms
apk add --no-cache librenms

# Enable and start service
rc-update add librenms default
rc-service librenms start

# Verify installation
librenms --version
```

### openSUSE/SLES

```bash
# Install librenms
sudo zypper install -y librenms

# Enable and start service
sudo systemctl enable --now librenms

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
librenms --version
```

### macOS

```bash
# Using Homebrew
brew install librenms

# Start service
brew services start librenms

# Verify installation
librenms --version
```

### FreeBSD

```bash
# Using pkg
pkg install librenms

# Enable in rc.conf
echo 'librenms_enable="YES"' >> /etc/rc.conf

# Start service
service librenms start

# Verify installation
librenms --version
```

### Windows

```bash
# Using Chocolatey
choco install librenms

# Or using Scoop
scoop install librenms

# Verify installation
librenms --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/librenms

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
librenms --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable librenms

# Start service
sudo systemctl start librenms

# Stop service
sudo systemctl stop librenms

# Restart service
sudo systemctl restart librenms

# Check status
sudo systemctl status librenms

# View logs
sudo journalctl -u librenms -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add librenms default

# Start service
rc-service librenms start

# Stop service
rc-service librenms stop

# Restart service
rc-service librenms restart

# Check status
rc-service librenms status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'librenms_enable="YES"' >> /etc/rc.conf

# Start service
service librenms start

# Stop service
service librenms stop

# Restart service
service librenms restart

# Check status
service librenms status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start librenms
brew services stop librenms
brew services restart librenms

# Check status
brew services list | grep librenms
```

### Windows Service Manager

```powershell
# Start service
net start librenms

# Stop service
net stop librenms

# Using PowerShell
Start-Service librenms
Stop-Service librenms
Restart-Service librenms

# Check status
Get-Service librenms
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream librenms_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name librenms.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name librenms.example.com;

    ssl_certificate /etc/ssl/certs/librenms.example.com.crt;
    ssl_certificate_key /etc/ssl/private/librenms.example.com.key;

    location / {
        proxy_pass http://librenms_backend;
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
    ServerName librenms.example.com
    Redirect permanent / https://librenms.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName librenms.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/librenms.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/librenms.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend librenms_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/librenms.pem
    redirect scheme https if !{ ssl_fc }
    default_backend librenms_backend

backend librenms_backend
    balance roundrobin
    server librenms1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R librenms:librenms /etc/librenms
sudo chmod 750 /etc/librenms

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
sudo systemctl status librenms

# View logs
sudo journalctl -u librenms -f

# Monitor resource usage
top -p $(pgrep librenms)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/librenms"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/librenms-backup-$DATE.tar.gz" /etc/librenms /var/lib/librenms

echo "Backup completed: $BACKUP_DIR/librenms-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop librenms

# Restore from backup
tar -xzf /backup/librenms/librenms-backup-*.tar.gz -C /

# Start service
sudo systemctl start librenms
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u librenms -n 100
sudo tail -f /var/log/librenms/librenms.log

# Check configuration
librenms --version

# Check permissions
ls -la /etc/librenms
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
top -p $(pgrep librenms)

# Check disk I/O
iotop -p $(pgrep librenms)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  librenms:
    image: librenms:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/librenms
      - ./data:/var/lib/librenms
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update librenms

# Debian/Ubuntu
sudo apt update && sudo apt upgrade librenms

# Arch Linux
sudo pacman -Syu librenms

# Alpine Linux
apk update && apk upgrade librenms

# openSUSE
sudo zypper update librenms

# FreeBSD
pkg update && pkg upgrade librenms

# Always backup before updates
tar -czf /backup/librenms-pre-update-$(date +%Y%m%d).tar.gz /etc/librenms

# Restart after updates
sudo systemctl restart librenms
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/librenms

# Clean old logs
find /var/log/librenms -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/librenms
```

## Additional Resources

- Official Documentation: https://docs.librenms.org/
- GitHub Repository: https://github.com/librenms/librenms
- Community Forum: https://forum.librenms.org/
- Best Practices Guide: https://docs.librenms.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
