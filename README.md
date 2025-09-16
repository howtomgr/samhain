# samhain Installation Guide

samhain is a free and open-source integrity monitoring. Samhain provides host-based intrusion detection system

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
  - Storage: 2GB for data
  - Network: Client/server
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 49777 (default samhain port)
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

# Install samhain
sudo dnf install -y samhain

# Enable and start service
sudo systemctl enable --now samhain

# Configure firewall
sudo firewall-cmd --permanent --add-port=49777/tcp
sudo firewall-cmd --reload

# Verify installation
samhain --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install samhain
sudo apt install -y samhain

# Enable and start service
sudo systemctl enable --now samhain

# Configure firewall
sudo ufw allow 49777

# Verify installation
samhain --version
```

### Arch Linux

```bash
# Install samhain
sudo pacman -S samhain

# Enable and start service
sudo systemctl enable --now samhain

# Verify installation
samhain --version
```

### Alpine Linux

```bash
# Install samhain
apk add --no-cache samhain

# Enable and start service
rc-update add samhain default
rc-service samhain start

# Verify installation
samhain --version
```

### openSUSE/SLES

```bash
# Install samhain
sudo zypper install -y samhain

# Enable and start service
sudo systemctl enable --now samhain

# Configure firewall
sudo firewall-cmd --permanent --add-port=49777/tcp
sudo firewall-cmd --reload

# Verify installation
samhain --version
```

### macOS

```bash
# Using Homebrew
brew install samhain

# Start service
brew services start samhain

# Verify installation
samhain --version
```

### FreeBSD

```bash
# Using pkg
pkg install samhain

# Enable in rc.conf
echo 'samhain_enable="YES"' >> /etc/rc.conf

# Start service
service samhain start

# Verify installation
samhain --version
```

### Windows

```bash
# Using Chocolatey
choco install samhain

# Or using Scoop
scoop install samhain

# Verify installation
samhain --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/samhain

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
samhain --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable samhain

# Start service
sudo systemctl start samhain

# Stop service
sudo systemctl stop samhain

# Restart service
sudo systemctl restart samhain

# Check status
sudo systemctl status samhain

# View logs
sudo journalctl -u samhain -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add samhain default

# Start service
rc-service samhain start

# Stop service
rc-service samhain stop

# Restart service
rc-service samhain restart

# Check status
rc-service samhain status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'samhain_enable="YES"' >> /etc/rc.conf

# Start service
service samhain start

# Stop service
service samhain stop

# Restart service
service samhain restart

# Check status
service samhain status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start samhain
brew services stop samhain
brew services restart samhain

# Check status
brew services list | grep samhain
```

### Windows Service Manager

```powershell
# Start service
net start samhain

# Stop service
net stop samhain

# Using PowerShell
Start-Service samhain
Stop-Service samhain
Restart-Service samhain

# Check status
Get-Service samhain
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream samhain_backend {
    server 127.0.0.1:49777;
}

server {
    listen 80;
    server_name samhain.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name samhain.example.com;

    ssl_certificate /etc/ssl/certs/samhain.example.com.crt;
    ssl_certificate_key /etc/ssl/private/samhain.example.com.key;

    location / {
        proxy_pass http://samhain_backend;
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
    ServerName samhain.example.com
    Redirect permanent / https://samhain.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName samhain.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/samhain.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/samhain.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:49777/
    ProxyPassReverse / http://127.0.0.1:49777/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend samhain_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/samhain.pem
    redirect scheme https if !{ ssl_fc }
    default_backend samhain_backend

backend samhain_backend
    balance roundrobin
    server samhain1 127.0.0.1:49777 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R samhain:samhain /etc/samhain
sudo chmod 750 /etc/samhain

# Configure firewall
sudo firewall-cmd --permanent --add-port=49777/tcp
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
sudo systemctl status samhain

# View logs
sudo journalctl -u samhain -f

# Monitor resource usage
top -p $(pgrep samhain)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/samhain"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/samhain-backup-$DATE.tar.gz" /etc/samhain /var/lib/samhain

echo "Backup completed: $BACKUP_DIR/samhain-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop samhain

# Restore from backup
tar -xzf /backup/samhain/samhain-backup-*.tar.gz -C /

# Start service
sudo systemctl start samhain
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u samhain -n 100
sudo tail -f /var/log/samhain/samhain.log

# Check configuration
samhain --version

# Check permissions
ls -la /etc/samhain
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 49777

# Test connectivity
telnet localhost 49777

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep samhain)

# Check disk I/O
iotop -p $(pgrep samhain)

# Check connections
ss -an | grep 49777
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  samhain:
    image: samhain:latest
    ports:
      - "49777:49777"
    volumes:
      - ./config:/etc/samhain
      - ./data:/var/lib/samhain
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update samhain

# Debian/Ubuntu
sudo apt update && sudo apt upgrade samhain

# Arch Linux
sudo pacman -Syu samhain

# Alpine Linux
apk update && apk upgrade samhain

# openSUSE
sudo zypper update samhain

# FreeBSD
pkg update && pkg upgrade samhain

# Always backup before updates
tar -czf /backup/samhain-pre-update-$(date +%Y%m%d).tar.gz /etc/samhain

# Restart after updates
sudo systemctl restart samhain
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/samhain

# Clean old logs
find /var/log/samhain -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/samhain
```

## Additional Resources

- Official Documentation: https://docs.samhain.org/
- GitHub Repository: https://github.com/samhain/samhain
- Community Forum: https://forum.samhain.org/
- Best Practices Guide: https://docs.samhain.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
