# vector Installation Guide

vector is a free and open-source observability pipeline. Vector provides high-performance observability data pipeline

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
  - RAM: 1GB minimum
  - Storage: 10GB for buffers
  - Network: Various sources
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8686 (default vector port)
  - API on 8686
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

# Install vector
sudo dnf install -y vector

# Enable and start service
sudo systemctl enable --now vector

# Configure firewall
sudo firewall-cmd --permanent --add-port=8686/tcp
sudo firewall-cmd --reload

# Verify installation
vector --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install vector
sudo apt install -y vector

# Enable and start service
sudo systemctl enable --now vector

# Configure firewall
sudo ufw allow 8686

# Verify installation
vector --version
```

### Arch Linux

```bash
# Install vector
sudo pacman -S vector

# Enable and start service
sudo systemctl enable --now vector

# Verify installation
vector --version
```

### Alpine Linux

```bash
# Install vector
apk add --no-cache vector

# Enable and start service
rc-update add vector default
rc-service vector start

# Verify installation
vector --version
```

### openSUSE/SLES

```bash
# Install vector
sudo zypper install -y vector

# Enable and start service
sudo systemctl enable --now vector

# Configure firewall
sudo firewall-cmd --permanent --add-port=8686/tcp
sudo firewall-cmd --reload

# Verify installation
vector --version
```

### macOS

```bash
# Using Homebrew
brew install vector

# Start service
brew services start vector

# Verify installation
vector --version
```

### FreeBSD

```bash
# Using pkg
pkg install vector

# Enable in rc.conf
echo 'vector_enable="YES"' >> /etc/rc.conf

# Start service
service vector start

# Verify installation
vector --version
```

### Windows

```bash
# Using Chocolatey
choco install vector

# Or using Scoop
scoop install vector

# Verify installation
vector --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/vector

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
vector --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable vector

# Start service
sudo systemctl start vector

# Stop service
sudo systemctl stop vector

# Restart service
sudo systemctl restart vector

# Check status
sudo systemctl status vector

# View logs
sudo journalctl -u vector -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add vector default

# Start service
rc-service vector start

# Stop service
rc-service vector stop

# Restart service
rc-service vector restart

# Check status
rc-service vector status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'vector_enable="YES"' >> /etc/rc.conf

# Start service
service vector start

# Stop service
service vector stop

# Restart service
service vector restart

# Check status
service vector status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start vector
brew services stop vector
brew services restart vector

# Check status
brew services list | grep vector
```

### Windows Service Manager

```powershell
# Start service
net start vector

# Stop service
net stop vector

# Using PowerShell
Start-Service vector
Stop-Service vector
Restart-Service vector

# Check status
Get-Service vector
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream vector_backend {
    server 127.0.0.1:8686;
}

server {
    listen 80;
    server_name vector.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name vector.example.com;

    ssl_certificate /etc/ssl/certs/vector.example.com.crt;
    ssl_certificate_key /etc/ssl/private/vector.example.com.key;

    location / {
        proxy_pass http://vector_backend;
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
    ServerName vector.example.com
    Redirect permanent / https://vector.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName vector.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/vector.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/vector.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8686/
    ProxyPassReverse / http://127.0.0.1:8686/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend vector_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/vector.pem
    redirect scheme https if !{ ssl_fc }
    default_backend vector_backend

backend vector_backend
    balance roundrobin
    server vector1 127.0.0.1:8686 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R vector:vector /etc/vector
sudo chmod 750 /etc/vector

# Configure firewall
sudo firewall-cmd --permanent --add-port=8686/tcp
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
sudo systemctl status vector

# View logs
sudo journalctl -u vector -f

# Monitor resource usage
top -p $(pgrep vector)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/vector"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/vector-backup-$DATE.tar.gz" /etc/vector /var/lib/vector

echo "Backup completed: $BACKUP_DIR/vector-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop vector

# Restore from backup
tar -xzf /backup/vector/vector-backup-*.tar.gz -C /

# Start service
sudo systemctl start vector
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u vector -n 100
sudo tail -f /var/log/vector/vector.log

# Check configuration
vector --version

# Check permissions
ls -la /etc/vector
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8686

# Test connectivity
telnet localhost 8686

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep vector)

# Check disk I/O
iotop -p $(pgrep vector)

# Check connections
ss -an | grep 8686
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  vector:
    image: vector:latest
    ports:
      - "8686:8686"
    volumes:
      - ./config:/etc/vector
      - ./data:/var/lib/vector
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update vector

# Debian/Ubuntu
sudo apt update && sudo apt upgrade vector

# Arch Linux
sudo pacman -Syu vector

# Alpine Linux
apk update && apk upgrade vector

# openSUSE
sudo zypper update vector

# FreeBSD
pkg update && pkg upgrade vector

# Always backup before updates
tar -czf /backup/vector-pre-update-$(date +%Y%m%d).tar.gz /etc/vector

# Restart after updates
sudo systemctl restart vector
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/vector

# Clean old logs
find /var/log/vector -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/vector
```

## Additional Resources

- Official Documentation: https://docs.vector.org/
- GitHub Repository: https://github.com/vector/vector
- Community Forum: https://forum.vector.org/
- Best Practices Guide: https://docs.vector.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
