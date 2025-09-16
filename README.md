# nats Installation Guide

nats is a free and open-source cloud native messaging system. NATS provides publish-subscribe, request-reply, and queue groups with at-most-once delivery, serving as a lightweight alternative to RabbitMQ or Kafka

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
  - Storage: 1GB for installation
  - Network: TCP for client connections
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 4222 (default nats port)
  - Port 8222 for monitoring
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

# Install nats
sudo dnf install -y nats-server

# Enable and start service
sudo systemctl enable --now nats

# Configure firewall
sudo firewall-cmd --permanent --add-port=4222/tcp
sudo firewall-cmd --reload

# Verify installation
nats-server --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install nats
sudo apt install -y nats-server

# Enable and start service
sudo systemctl enable --now nats

# Configure firewall
sudo ufw allow 4222

# Verify installation
nats-server --version
```

### Arch Linux

```bash
# Install nats
sudo pacman -S nats-server

# Enable and start service
sudo systemctl enable --now nats

# Verify installation
nats-server --version
```

### Alpine Linux

```bash
# Install nats
apk add --no-cache nats-server

# Enable and start service
rc-update add nats default
rc-service nats start

# Verify installation
nats-server --version
```

### openSUSE/SLES

```bash
# Install nats
sudo zypper install -y nats-server

# Enable and start service
sudo systemctl enable --now nats

# Configure firewall
sudo firewall-cmd --permanent --add-port=4222/tcp
sudo firewall-cmd --reload

# Verify installation
nats-server --version
```

### macOS

```bash
# Using Homebrew
brew install nats-server

# Start service
brew services start nats-server

# Verify installation
nats-server --version
```

### FreeBSD

```bash
# Using pkg
pkg install nats-server

# Enable in rc.conf
echo 'nats_enable="YES"' >> /etc/rc.conf

# Start service
service nats start

# Verify installation
nats-server --version
```

### Windows

```bash
# Using Chocolatey
choco install nats-server

# Or using Scoop
scoop install nats-server

# Verify installation
nats-server --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/nats-server

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
nats-server --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable nats

# Start service
sudo systemctl start nats

# Stop service
sudo systemctl stop nats

# Restart service
sudo systemctl restart nats

# Check status
sudo systemctl status nats

# View logs
sudo journalctl -u nats -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add nats default

# Start service
rc-service nats start

# Stop service
rc-service nats stop

# Restart service
rc-service nats restart

# Check status
rc-service nats status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'nats_enable="YES"' >> /etc/rc.conf

# Start service
service nats start

# Stop service
service nats stop

# Restart service
service nats restart

# Check status
service nats status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start nats-server
brew services stop nats-server
brew services restart nats-server

# Check status
brew services list | grep nats-server
```

### Windows Service Manager

```powershell
# Start service
net start nats

# Stop service
net stop nats

# Using PowerShell
Start-Service nats
Stop-Service nats
Restart-Service nats

# Check status
Get-Service nats
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream nats-server_backend {
    server 127.0.0.1:4222;
}

server {
    listen 80;
    server_name nats-server.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name nats-server.example.com;

    ssl_certificate /etc/ssl/certs/nats-server.example.com.crt;
    ssl_certificate_key /etc/ssl/private/nats-server.example.com.key;

    location / {
        proxy_pass http://nats-server_backend;
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
    ServerName nats-server.example.com
    Redirect permanent / https://nats-server.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName nats-server.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/nats-server.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/nats-server.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:4222/
    ProxyPassReverse / http://127.0.0.1:4222/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend nats-server_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/nats-server.pem
    redirect scheme https if !{ ssl_fc }
    default_backend nats-server_backend

backend nats-server_backend
    balance roundrobin
    server nats-server1 127.0.0.1:4222 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R nats-server:nats-server /etc/nats-server
sudo chmod 750 /etc/nats-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=4222/tcp
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
sudo systemctl status nats

# View logs
sudo journalctl -u nats -f

# Monitor resource usage
top -p $(pgrep nats-server)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/nats-server"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/nats-server-backup-$DATE.tar.gz" /etc/nats-server /var/lib/nats-server

echo "Backup completed: $BACKUP_DIR/nats-server-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop nats

# Restore from backup
tar -xzf /backup/nats-server/nats-server-backup-*.tar.gz -C /

# Start service
sudo systemctl start nats
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u nats -n 100
sudo tail -f /var/log/nats-server/nats-server.log

# Check configuration
nats-server --version

# Check permissions
ls -la /etc/nats-server
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 4222

# Test connectivity
telnet localhost 4222

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep nats-server)

# Check disk I/O
iotop -p $(pgrep nats-server)

# Check connections
ss -an | grep 4222
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  nats-server:
    image: nats-server:latest
    ports:
      - "4222:4222"
    volumes:
      - ./config:/etc/nats-server
      - ./data:/var/lib/nats-server
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update nats-server

# Debian/Ubuntu
sudo apt update && sudo apt upgrade nats-server

# Arch Linux
sudo pacman -Syu nats-server

# Alpine Linux
apk update && apk upgrade nats-server

# openSUSE
sudo zypper update nats-server

# FreeBSD
pkg update && pkg upgrade nats-server

# Always backup before updates
tar -czf /backup/nats-server-pre-update-$(date +%Y%m%d).tar.gz /etc/nats-server

# Restart after updates
sudo systemctl restart nats
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/nats-server

# Clean old logs
find /var/log/nats-server -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/nats-server
```

## Additional Resources

- Official Documentation: https://docs.nats-server.org/
- GitHub Repository: https://github.com/nats-server/nats-server
- Community Forum: https://forum.nats-server.org/
- Best Practices Guide: https://docs.nats-server.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
