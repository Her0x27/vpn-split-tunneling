# VPN Split Tunneling

A sophisticated solution for domain-based VPN split tunneling with nftables filtering, automatic failover protection, and seamless traffic routing.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Domain-Based Routing](#domain-based-routing)
- [nftables Filtering](#nftables-filtering)
- [Failover Protection](#failover-protection)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Overview

VPN Split Tunneling is an advanced traffic routing solution that allows you to selectively route specific domains through a VPN tunnel while keeping other traffic on your default connection. Using kernel-level nftables filtering and intelligent failover mechanisms, it provides:

- **Domain-based routing**: Automatically route traffic for specified domains through VPN
- **Transparent filtering**: Kernel-level packet filtering with nftables
- **Failover protection**: Automatic recovery if VPN connection drops
- **Low latency**: Minimal overhead with kernel-space processing
- **Easy configuration**: Simple YAML-based configuration files

## Features

### 🎯 Domain-Based Routing
- Route specific domains through VPN tunnel automatically
- Support for wildcard domain matching
- Real-time DNS resolution integration
- Per-domain routing policies

### 🔥 nftables Filtering
- Kernel-space packet filtering for optimal performance
- Stateful connection tracking
- Dynamic rule management
- Support for IPv4 and IPv6

### 🛡️ Failover Protection
- Automatic VPN disconnection detection
- Graceful traffic rerouting on failure
- Health check mechanisms
- Connection recovery strategies
- Kill switch functionality

### ⚡ Performance
- Minimal CPU overhead
- Low-latency packet processing
- Efficient memory usage
- Real-time traffic monitoring

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Applications                         │
└────────────┬────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────┐
│            DNS Resolution / Domain Detection            │
│        (Identifies domains requiring VPN routing)       │
└────────────┬────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────┐
│              nftables Filtering Rules                   │
│         (Routes traffic to VPN or default gateway)      │
└────────────┬────────────────────────────────────────────┘
             │
      ┌──────┴──────┐
      ▼             ▼
  ┌────────┐   ┌──────────┐
  │  VPN   │   │ Default  │
  │ Tunnel │   │ Gateway  │
  └────────┘   └──────────┘
      │             │
      └──────┬──────┘
             ▼
      ┌──────────────┐
      │  Internet    │
      └──────────────┘
```

## Prerequisites

### System Requirements

- **Linux kernel**: 4.10 or higher (for nftables support)
- **OS**: Any Linux distribution with:
  - nftables installed and enabled
  - systemd (recommended for service management)
  - Python 3.8+ (for management scripts)

### Required Packages

```bash
# Ubuntu/Debian
sudo apt-get install nftables openvpn python3 python3-pip

# RHEL/CentOS
sudo dnf install nftables openvpn python3 python3-pip

# Arch
sudo pacman -S nftables openvpn python python-pip
```

### Required Python Packages

```bash
sudo pip3 install PyYAML dnspython iproute2
```

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/Her0x27/vpn-split-tunneling.git
cd vpn-split-tunneling
```

### 2. Install

```bash
sudo bash install.sh
```

### 3. Basic Configuration

Create a configuration file at `/etc/vpn-split-tunneling/config.yaml`:

```yaml
# VPN Configuration
vpn:
  name: "myvpn"
  type: "openvpn"
  config_path: "/etc/openvpn/client.conf"
  interface: "tun0"
  
# Domains to route through VPN
domains:
  - "*.torrent.example.com"
  - "private-service.local"
  - "vpn-only.org"
  - "restricted-region.net"

# Failover Settings
failover:
  enabled: true
  check_interval: 30
  max_retries: 3
  kill_switch: true
  
# nftables Rules
nftables:
  priority: 100
  enable_ipv6: true
```

### 4. Start the Service

```bash
sudo systemctl start vpn-split-tunneling
sudo systemctl enable vpn-split-tunneling
```

### 5. Verify Status

```bash
sudo systemctl status vpn-split-tunneling
journalctl -u vpn-split-tunneling -f
```

## Configuration

### Main Configuration File

The main configuration uses YAML format. Here's a comprehensive example:

```yaml
# VPN Settings
vpn:
  # VPN connection name
  name: "production-vpn"
  
  # VPN type: openvpn, wireguard, ipsec, etc.
  type: "openvpn"
  
  # Path to VPN configuration
  config_path: "/etc/openvpn/client.ovpn"
  
  # VPN tunnel interface name
  interface: "tun0"
  
  # VPN credentials (optional, can be prompted)
  username: "vpn_user"
  password: "${VPN_PASSWORD}"  # Use environment variables
  
  # Connection timeout in seconds
  timeout: 30

# DNS Configuration
dns:
  # DNS servers to use for VPN queries
  vpn_servers:
    - "10.0.0.1"
    - "10.0.0.2"
  
  # DNS servers for default gateway
  default_servers:
    - "8.8.8.8"
    - "8.8.4.4"
  
  # Enable DNS over HTTPS
  dohs: false

# Domains to route through VPN
domains:
  # Exact domain match
  - "company.com"
  
  # Wildcard subdomains
  - "*.internal.company.com"
  
  # Regional services
  - "*.region.service.com"
  
  # Specific hostnames
  - "vpn-database.prod"

# Failover Configuration
failover:
  # Enable failover protection
  enabled: true
  
  # Health check interval (seconds)
  check_interval: 30
  
  # VPN connectivity check method
  check_method: "ping"  # ping, http, dns
  check_target: "10.0.0.1"  # Gateway or service IP
  
  # Number of failed checks before failover
  failure_threshold: 3
  
  # Maximum retry attempts
  max_retries: 5
  
  # Enable kill switch (block all traffic if VPN fails)
  kill_switch: true
  
  # Actions on VPN failure
  on_failure:
    - "log"
    - "alert"
    - "disconnect"
    - "restore_default"

# nftables Configuration
nftables:
  # Rule priority (lower = higher priority)
  priority: 100
  
  # Table name
  table: "filter_vpn"
  
  # Enable IPv6 support
  enable_ipv6: true
  
  # Connection tracking
  conntrack: true
  
  # Log level: off, low, medium, high
  log_level: "medium"
  
  # Rate limiting
  rate_limit:
    enabled: true
    packets_per_second: 10000

# Logging Configuration
logging:
  # Log level: DEBUG, INFO, WARNING, ERROR, CRITICAL
  level: "INFO"
  
  # Log file path
  file: "/var/log/vpn-split-tunneling.log"
  
  # Maximum log file size (MB)
  max_size: 100
  
  # Number of log files to keep
  backup_count: 5
  
  # Log to syslog
  syslog: true

# Advanced Settings
advanced:
  # Monitor only specific interfaces
  monitor_interfaces:
    - "eth0"
    - "wlan0"
  
  # Exclude IPs from VPN routing
  exclude_ips:
    - "192.168.1.0/24"
    - "10.0.0.0/8"
  
  # Enable statistics collection
  statistics: true
  
  # Statistics update interval (seconds)
  stats_interval: 60
```

## Domain-Based Routing

### How It Works

1. **DNS Interception**: The system monitors DNS queries
2. **Domain Matching**: Compares resolved IPs against configured domain list
3. **Rule Generation**: Creates nftables rules for matching traffic
4. **Traffic Direction**: Routes matched traffic through VPN tunnel

### Domain Matching Patterns

```yaml
domains:
  # Exact match
  - "example.com"
  
  # Subdomain wildcard
  - "*.example.com"      # matches api.example.com, app.example.com
  
  # Multi-level wildcard
  - "*.*.example.com"    # matches a.b.example.com
  
  # Regex pattern (with regex: prefix)
  - "regex:vpn-.*\\.company\\.com"
  
  # IP CIDR ranges
  - "ip:10.20.0.0/16"
```

### Dynamic Domain Addition

Add domains at runtime without restarting:

```bash
sudo vpn-split-tunneling add-domain "newdomain.com"
sudo vpn-split-tunneling add-domain "*.newdomain.com"
```

### List Current Domains

```bash
sudo vpn-split-tunneling list-domains
```

### Remove Domains

```bash
sudo vpn-split-tunneling remove-domain "olddomain.com"
```

## nftables Filtering

### How It Works

The system uses nftables (netfilter) for kernel-space packet filtering:

```
Incoming Packet
      │
      ▼
┌─────────────────┐
│ Check nftables  │
│   rules for     │
│  destination IP │
└────────┬────────┘
         │
    ┌────┴────┐
    │          │
    ▼          ▼
 VPN Route  Default Route
```

### Generated Rules

The system automatically generates rules like:

```nftables
table inet filter_vpn {
  chain output {
    # Route specific IPs through VPN
    ip daddr 93.184.216.34 mark set 100
    
    # Route with DNS domain matching
    ip daddr 203.0.113.0/24 mark set 100
  }
  
  chain mangle {
    # Mark packets for VPN routing
    mark 100 oif tun0 accept
  }
}
```

### View Current Rules

```bash
sudo nft list table inet filter_vpn
```

### Manual Rule Inspection

```bash
sudo nft list ruleset | grep -A 10 filter_vpn
```

## Failover Protection

### Health Checking

The system continuously monitors VPN connectivity:

```yaml
failover:
  check_method: "ping"
  check_target: "10.0.0.1"        # VPN gateway
  check_interval: 30              # Every 30 seconds
  failure_threshold: 3            # Trigger after 3 failures
```

### Check Methods

1. **ping**: ICMP ping to target
   ```yaml
   check_method: "ping"
   check_target: "10.0.0.1"
   ```

2. **http**: HTTP/HTTPS request
   ```yaml
   check_method: "http"
   check_target: "http://10.0.0.1/health"
   ```

3. **dns**: DNS query resolution
   ```yaml
   check_method: "dns"
   check_target: "vpn-internal.local"
   ```

### Failover Actions

When VPN fails:

```yaml
on_failure:
  - "log"              # Log the event
  - "alert"            # Send alert notification
  - "disconnect"       # Disconnect VPN
  - "restore_default"  # Route all traffic to default gateway
  - "block"            # Block all traffic (kill switch)
```

### Kill Switch

Prevents leaks if VPN unexpectedly disconnects:

```bash
# Enable kill switch
sudo vpn-split-tunneling kill-switch enable

# Disable kill switch
sudo vpn-split-tunneling kill-switch disable

# Check status
sudo vpn-split-tunneling kill-switch status
```

### Recovery

Automatic recovery when VPN reconnects:

```yaml
failover:
  max_retries: 5
  recovery_wait: 10    # Wait 10 seconds before retry
```

## Usage

### Service Management

```bash
# Start service
sudo systemctl start vpn-split-tunneling

# Stop service
sudo systemctl stop vpn-split-tunneling

# Restart service
sudo systemctl restart vpn-split-tunneling

# Enable on boot
sudo systemctl enable vpn-split-tunneling

# Disable on boot
sudo systemctl disable vpn-split-tunneling

# Check status
sudo systemctl status vpn-split-tunneling

# View logs
journalctl -u vpn-split-tunneling -f
```

### Command Line Interface

```bash
# Show current status
sudo vpn-split-tunneling status

# Show statistics
sudo vpn-split-tunneling stats

# List active rules
sudo vpn-split-tunneling list-rules

# List configured domains
sudo vpn-split-tunneling list-domains

# Add domain
sudo vpn-split-tunneling add-domain "example.com"

# Remove domain
sudo vpn-split-tunneling remove-domain "example.com"

# Enable kill switch
sudo vpn-split-tunneling kill-switch enable

# Test VPN connectivity
sudo vpn-split-tunneling test-vpn

# Reload configuration
sudo vpn-split-tunneling reload

# Reset all rules
sudo vpn-split-tunneling reset
```

### Monitoring

```bash
# Real-time traffic statistics
sudo vpn-split-tunneling stats --follow

# Monitor specific domain
sudo vpn-split-tunneling monitor "example.com"

# Export statistics
sudo vpn-split-tunneling export-stats --output stats.json
```

### Testing

```bash
# Test DNS resolution
dig @8.8.8.8 example.com
dig @10.0.0.1 example.com

# Test connectivity through VPN
curl --interface tun0 https://example.com

# Check routing for IP
ip route get 93.184.216.34

# Verify nftables rules
sudo nft list ruleset | grep -A 20 filter_vpn
```

## Troubleshooting

### VPN Not Connecting

```bash
# Check VPN service logs
journalctl -u openvpn-client -n 50

# Test VPN configuration
sudo openvpn --config /etc/openvpn/client.conf --test-crypto

# Check nftables rules
sudo nft list table inet filter_vpn

# Verify interface is up
ip link show tun0
```

### Traffic Not Routing Through VPN

```bash
# Check if domain is in configuration
sudo vpn-split-tunneling list-domains | grep "example.com"

# Resolve domain and check IP
nslookup example.com

# Check nftables rules for the IP
sudo nft list table inet filter_vpn | grep "YOUR_IP"

# Test routing
ip route get 93.184.216.34

# Check firewall rules
sudo nft list ruleset | head -50
```

### High CPU/Memory Usage

```bash
# Check process resource usage
top -p $(pidof vpn-split-tunneling)

# Check nftables rule count
sudo nft list ruleset | wc -l

# Monitor in real-time
sudo vpn-split-tunneling stats --follow

# Consider reducing domain list or check frequency
```

### Failover Not Triggering

```bash
# Check failover configuration
grep -A 10 "failover:" /etc/vpn-split-tunneling/config.yaml

# Manually test health check
ping -c 1 10.0.0.1

# Check failover logs
journalctl -u vpn-split-tunneling | grep failover

# Verify kill switch state
sudo vpn-split-tunneling kill-switch status
```

### DNS Issues

```bash
# Check DNS configuration
cat /etc/vpn-split-tunneling/config.yaml | grep -A 5 "dns:"

# Test DNS resolution
dig @8.8.8.8 example.com
dig @10.0.0.1 example.com

# Check systemd-resolved status
systemctl status systemd-resolved

# Flush DNS cache
sudo systemd-resolve --flush-caches
```

### Permission Issues

```bash
# Verify running as root
whoami

# Check file permissions
ls -la /etc/vpn-split-tunneling/

# Fix permissions
sudo chown -R root:root /etc/vpn-split-tunneling/
sudo chmod 600 /etc/vpn-split-tunneling/config.yaml
```

## Advanced Topics

### Custom nftables Rules

Extend the default rules:

```bash
# Add custom rule
sudo nft add rule inet filter_vpn output \
  ip daddr 203.0.113.0/24 mark set 100

# List rules
sudo nft list table inet filter_vpn
```

### Multiple VPN Connections

Configure failover to secondary VPN:

```yaml
vpn:
  primary:
    name: "vpn1"
    interface: "tun0"
  secondary:
    name: "vpn2"
    interface: "tun1"
```

### Cgroups Integration

Limit resource usage:

```yaml
advanced:
  cgroups:
    enabled: true
    cpu_quota: 50
    memory_limit: "256M"
```

### Metrics Export

```bash
# Export Prometheus metrics
sudo vpn-split-tunneling export-metrics --format prometheus
```

## Performance Optimization

### For High-Traffic Environments

```yaml
nftables:
  conntrack: true
  rate_limit:
    enabled: true
    packets_per_second: 50000

advanced:
  statistics: false  # Disable if not needed
```

### For Low-Resource Systems

```yaml
failover:
  check_interval: 60  # Increase interval
  
nftables:
  log_level: "off"    # Disable logging
```

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

- **Issues**: [GitHub Issues](https://github.com/Her0x27/vpn-split-tunneling/issues)
- **Documentation**: Full docs in `/docs` directory
- **Examples**: Configuration examples in `/examples` directory

## Disclaimer

This tool is provided as-is for technical and educational purposes. Users are responsible for complying with all applicable laws and regulations in their jurisdiction. Unauthorized access to computer networks is illegal.

---

**Last Updated**: December 27, 2025
**Maintained by**: Her0x27
