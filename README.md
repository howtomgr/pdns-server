# powerdns Installation Guide

powerdns is a free and open-source modern authoritative DNS server. PowerDNS provides high-performance DNS with database backends and API, serving as a modern alternative to BIND

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
  - Storage: 200MB for installation
  - Network: UDP/TCP port 53
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 53 (default powerdns port)
  - Port 8081 for API
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

# Install powerdns
sudo dnf install -y pdns-server

# Enable and start service
sudo systemctl enable --now pdns

# Configure firewall
sudo firewall-cmd --permanent --add-port=53/tcp
sudo firewall-cmd --reload

# Verify installation
pdns_server --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install powerdns
sudo apt install -y pdns-server

# Enable and start service
sudo systemctl enable --now pdns

# Configure firewall
sudo ufw allow 53

# Verify installation
pdns_server --version
```

### Arch Linux

```bash
# Install powerdns
sudo pacman -S pdns-server

# Enable and start service
sudo systemctl enable --now pdns

# Verify installation
pdns_server --version
```

### Alpine Linux

```bash
# Install powerdns
apk add --no-cache pdns-server

# Enable and start service
rc-update add pdns default
rc-service pdns start

# Verify installation
pdns_server --version
```

### openSUSE/SLES

```bash
# Install powerdns
sudo zypper install -y pdns-server

# Enable and start service
sudo systemctl enable --now pdns

# Configure firewall
sudo firewall-cmd --permanent --add-port=53/tcp
sudo firewall-cmd --reload

# Verify installation
pdns_server --version
```

### macOS

```bash
# Using Homebrew
brew install pdns-server

# Start service
brew services start pdns-server

# Verify installation
pdns_server --version
```

### FreeBSD

```bash
# Using pkg
pkg install pdns-server

# Enable in rc.conf
echo 'pdns_enable="YES"' >> /etc/rc.conf

# Start service
service pdns start

# Verify installation
pdns_server --version
```

### Windows

```bash
# Using Chocolatey
choco install pdns-server

# Or using Scoop
scoop install pdns-server

# Verify installation
pdns_server --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/pdns-server

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
pdns_server --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable pdns

# Start service
sudo systemctl start pdns

# Stop service
sudo systemctl stop pdns

# Restart service
sudo systemctl restart pdns

# Check status
sudo systemctl status pdns

# View logs
sudo journalctl -u pdns -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add pdns default

# Start service
rc-service pdns start

# Stop service
rc-service pdns stop

# Restart service
rc-service pdns restart

# Check status
rc-service pdns status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'pdns_enable="YES"' >> /etc/rc.conf

# Start service
service pdns start

# Stop service
service pdns stop

# Restart service
service pdns restart

# Check status
service pdns status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start pdns-server
brew services stop pdns-server
brew services restart pdns-server

# Check status
brew services list | grep pdns-server
```

### Windows Service Manager

```powershell
# Start service
net start pdns

# Stop service
net stop pdns

# Using PowerShell
Start-Service pdns
Stop-Service pdns
Restart-Service pdns

# Check status
Get-Service pdns
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream pdns-server_backend {
    server 127.0.0.1:53;
}

server {
    listen 80;
    server_name pdns-server.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name pdns-server.example.com;

    ssl_certificate /etc/ssl/certs/pdns-server.example.com.crt;
    ssl_certificate_key /etc/ssl/private/pdns-server.example.com.key;

    location / {
        proxy_pass http://pdns-server_backend;
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
    ServerName pdns-server.example.com
    Redirect permanent / https://pdns-server.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName pdns-server.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/pdns-server.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/pdns-server.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:53/
    ProxyPassReverse / http://127.0.0.1:53/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend pdns-server_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/pdns-server.pem
    redirect scheme https if !{ ssl_fc }
    default_backend pdns-server_backend

backend pdns-server_backend
    balance roundrobin
    server pdns-server1 127.0.0.1:53 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R pdns-server:pdns-server /etc/pdns-server
sudo chmod 750 /etc/pdns-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=53/tcp
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
sudo systemctl status pdns

# View logs
sudo journalctl -u pdns -f

# Monitor resource usage
top -p $(pgrep pdns-server)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/pdns-server"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/pdns-server-backup-$DATE.tar.gz" /etc/pdns-server /var/lib/pdns-server

echo "Backup completed: $BACKUP_DIR/pdns-server-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop pdns

# Restore from backup
tar -xzf /backup/pdns-server/pdns-server-backup-*.tar.gz -C /

# Start service
sudo systemctl start pdns
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u pdns -n 100
sudo tail -f /var/log/pdns-server/pdns-server.log

# Check configuration
pdns_server --version

# Check permissions
ls -la /etc/pdns-server
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 53

# Test connectivity
telnet localhost 53

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep pdns-server)

# Check disk I/O
iotop -p $(pgrep pdns-server)

# Check connections
ss -an | grep 53
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  pdns-server:
    image: pdns-server:latest
    ports:
      - "53:53"
    volumes:
      - ./config:/etc/pdns-server
      - ./data:/var/lib/pdns-server
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update pdns-server

# Debian/Ubuntu
sudo apt update && sudo apt upgrade pdns-server

# Arch Linux
sudo pacman -Syu pdns-server

# Alpine Linux
apk update && apk upgrade pdns-server

# openSUSE
sudo zypper update pdns-server

# FreeBSD
pkg update && pkg upgrade pdns-server

# Always backup before updates
tar -czf /backup/pdns-server-pre-update-$(date +%Y%m%d).tar.gz /etc/pdns-server

# Restart after updates
sudo systemctl restart pdns
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/pdns-server

# Clean old logs
find /var/log/pdns-server -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/pdns-server
```

## Additional Resources

- Official Documentation: https://docs.pdns-server.org/
- GitHub Repository: https://github.com/pdns-server/pdns-server
- Community Forum: https://forum.pdns-server.org/
- Best Practices Guide: https://docs.pdns-server.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
