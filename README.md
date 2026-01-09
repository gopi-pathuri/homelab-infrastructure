# Enterprise Home Lab Infrastructure

A production-grade home lab environment simulating enterprise network architecture with VMware ESXi virtualization, pfSense firewall, segmented VLANs, intrusion detection, VPN capabilities, and comprehensive monitoring.

## 🏗️ Architecture Overview

This home lab replicates enterprise network infrastructure to develop and demonstrate skills in:
- **Virtualization**: VMware ESXi hypervisor management
- **Network Security**: Firewall rules, IDS/IPS, VPN tunnels
- **Network Segmentation**: VLAN design and implementation
- **Monitoring**: Prometheus and Grafana for infrastructure observability
- **Traffic Engineering**: QoS and traffic shaping policies

## 📊 Network Topology

```
Internet
   |
[pfSense Firewall/Router]
   |
   +-- VLAN 10 (Management)    - 10.0.10.0/24
   +-- VLAN 20 (Internal/LAN)  - 10.0.20.0/24
   +-- VLAN 30 (DMZ)           - 10.0.30.0/24
   +-- VLAN 40 (Guest)         - 10.0.40.0/24
   +-- VLAN 99 (ESXi Hosts)    - 10.0.99.0/24
```

## 🖥️ Infrastructure Components

### Hardware Specifications
- **Server**: Dell PowerEdge R720 (or equivalent)
  - CPU: 2x Intel Xeon E5-2670 (16 cores, 32 threads)
  - RAM: 128GB DDR3 ECC
  - Storage: 4x 1TB SAS (RAID 10)
  - Network: Quad-port Intel I350 NIC

### Virtualization Platform
- **VMware ESXi 7.0+**
  - Distributed Resource Scheduler (DRS)
  - High Availability (HA) enabled
  - vMotion configured
  - Resource pools for workload segregation

### Network Infrastructure
- **pfSense 2.7+** (Virtual Appliance)
  - 4 vCPUs, 8GB RAM
  - Multi-WAN failover
  - IDS/IPS (Suricata)
  - OpenVPN & IPsec VPN
  - Traffic shaping and QoS

## 🔒 Security Implementation

### Firewall Rules

#### Management VLAN (10)
- SSH access to ESXi hosts
- vCenter management traffic
- Monitoring system access
- Deny all other inbound traffic

#### Internal VLAN (20)
- Outbound internet access (NAT)
- Access to DMZ services
- Blocked from Management VLAN

#### DMZ VLAN (30)
- Public-facing services
- Port forwarding for web servers
- Restricted access to Internal network
- Monitored by IDS/IPS

#### Guest VLAN (40)
- Internet-only access
- Isolated from all other VLANs
- DHCP with short lease times
- Rate limiting applied

### IDS/IPS Configuration

**Suricata Engine**
- **Rulesets**: Emerging Threats Pro
- **Categories Enabled**:
  - Malware detection
  - Exploit attempts
  - Command and control traffic
  - Port scanning detection
  - SQL injection attempts
  - Cross-site scripting (XSS)

**Alert Actions**:
- Block malicious traffic automatically
- Log all events to syslog
- Email notifications for critical alerts
- Integration with Grafana dashboards

### VPN Implementation

#### OpenVPN (Remote Access)
- **Authentication**: Certificate-based
- **Encryption**: AES-256-GCM
- **Tunnel**: 10.0.50.0/24
- **Features**:
  - Split tunneling option
  - DNS pushed to clients
  - 2FA with Google Authenticator
  - Client config generator

#### IPsec (Site-to-Site)
- **Phase 1**: IKEv2, AES-256, SHA256
- **Phase 2**: AES-256-GCM, PFS enabled
- **Use Case**: Connect to cloud VPC

## 📡 Virtual Machine Inventory

| VM Name | OS | vCPU | RAM | Storage | VLAN | Purpose |
|---------|-------|------|-----|---------|------|----------|
| pfSense-FW01 | pfSense 2.7 | 4 | 8GB | 32GB | Trunk | Firewall/Router |
| DC01 | Windows Server 2022 | 2 | 4GB | 60GB | 10 | Domain Controller |
| Prometheus-01 | Ubuntu 22.04 | 2 | 4GB | 40GB | 10 | Monitoring |
| Grafana-01 | Ubuntu 22.04 | 2 | 4GB | 40GB | 10 | Dashboards |
| WebServer-01 | Ubuntu 22.04 | 2 | 4GB | 40GB | 30 | DMZ Web Server |
| Workstation-01 | Windows 11 | 4 | 8GB | 100GB | 20 | Testing/Development |

## 📈 Monitoring & Observability

### Prometheus Configuration

**Data Sources**:
- pfSense exporter (firewall metrics)
- Node exporter (system metrics)
- ESXi exporter (virtualization metrics)
- SNMP exporter (network equipment)

**Metrics Collected**:
- CPU, memory, disk utilization
- Network throughput and packet rates
- Firewall state table usage
- VPN tunnel status
- IDS/IPS alert counts
- VM resource consumption

**Retention**: 30 days of metrics data

### Grafana Dashboards

1. **Network Overview**
   - Real-time bandwidth utilization
   - Top talkers by IP and port
   - Firewall rule hit counts
   - VPN connection status

2. **Security Dashboard**
   - IDS/IPS alerts by severity
   - Blocked connections heatmap
   - Geographic threat map
   - Failed authentication attempts

3. **Infrastructure Health**
   - ESXi host performance
   - VM resource allocation
   - Storage I/O metrics
   - Network latency graphs

4. **Application Monitoring**
   - Service uptime status
   - Response time metrics
   - Error rate tracking
   - Custom application metrics

### Alerting Configuration

**Alert Channels**:
- Email notifications
- Slack webhooks
- PagerDuty integration

**Alert Rules**:
- High CPU usage (>85% for 5 minutes)
- Memory pressure (>90%)
- Disk space low (<15% free)
- Service down (HTTP 200 check fails)
- IDS critical alerts
- VPN tunnel down

## 🚀 Traffic Shaping & QoS

### Priority Queues

1. **VoIP/Real-time** (Priority 1)
   - 20% guaranteed bandwidth
   - Max latency: 50ms
   - Protocols: SIP, RTP

2. **Management** (Priority 2)
   - 15% guaranteed bandwidth
   - SSH, RDP, vCenter

3. **Business Applications** (Priority 3)
   - 40% guaranteed bandwidth
   - HTTP/HTTPS, Database traffic

4. **Bulk Transfer** (Priority 4)
   - 15% guaranteed bandwidth
   - FTP, Backup traffic

5. **Default/Guest** (Priority 5)
   - 10% guaranteed bandwidth
   - All other traffic

## 🔧 Configuration Management

### pfSense Backup Strategy
- **Automatic**: Weekly configuration backups
- **Manual**: Before any major changes
- **Storage**: Encrypted backup to NAS
- **Versioning**: Keep last 10 revisions

### ESXi Configuration
- **Host profiles**: Standardized settings
- **Distributed switches**: Consistent networking
- **Auto Deploy**: Rapid host provisioning

## 📚 Documentation Structure

```
├── docs/
│   ├── network/
│   │   ├── vlan-design.md
│   │   ├── ip-addressing.md
│   │   └── routing-policies.md
│   ├── security/
│   │   ├── firewall-rules.md
│   │   ├── ids-ips-tuning.md
│   │   └── vpn-setup.md
│   ├── monitoring/
│   │   ├── prometheus-setup.md
│   │   ├── grafana-dashboards.md
│   │   └── alerting-rules.md
│   └── virtualization/
│       ├── esxi-installation.md
│       ├── vm-templates.md
│       └── resource-allocation.md
├── configs/
│   ├── pfsense/
│   ├── prometheus/
│   └── grafana/
└── scripts/
    ├── backup/
    ├── monitoring/
    └── automation/
```

## 🎯 Key Achievements

✅ **Network Segmentation**: Implemented enterprise-grade VLAN architecture with proper isolation

✅ **Security Hardening**: Deployed multi-layered security with IDS/IPS, firewall rules, and VPN

✅ **High Availability**: Configured ESXi HA and vMotion for zero-downtime maintenance

✅ **Monitoring**: Built comprehensive observability with Prometheus and Grafana

✅ **Performance Optimization**: Implemented QoS and traffic shaping for optimal performance

✅ **Automation**: Created scripts for backup, monitoring, and configuration management

## 🔄 Ongoing Operations

### Maintenance Schedule
- **Daily**: Check monitoring dashboards and alerts
- **Weekly**: Review IDS/IPS logs and security events
- **Monthly**: Update firmware and patches
- **Quarterly**: Capacity planning and performance review

### Disaster Recovery
- VM backups: Daily incremental, weekly full
- Configuration backups: Automated to off-site storage
- Documentation: Version controlled in Git
- Recovery Time Objective (RTO): 4 hours
- Recovery Point Objective (RPO): 24 hours

## 💡 Skills Demonstrated

- **Virtualization**: VMware ESXi administration and optimization
- **Network Design**: VLAN segmentation and routing
- **Firewall Management**: pfSense advanced configuration
- **Security Operations**: IDS/IPS tuning and incident response
- **VPN Technologies**: OpenVPN and IPsec implementation
- **Monitoring**: Prometheus/Grafana deployment and dashboard creation
- **Traffic Engineering**: QoS policies and bandwidth management
- **Documentation**: Comprehensive technical documentation
- **Automation**: Scripting for operations and maintenance

## 🌐 Future Enhancements

- [ ] Implement Kubernetes cluster for container orchestration
- [ ] Add Suricata custom rule development
- [ ] Deploy ELK stack for log aggregation
- [ ] Integrate with cloud platforms (AWS/Azure)
- [ ] Implement zero-trust network architecture
- [ ] Add honeypot systems for threat research
- [ ] Develop Ansible playbooks for automation
- [ ] Create disaster recovery automation

## 📖 References & Resources

- **VMware Documentation**: [docs.vmware.com](https://docs.vmware.com)
- **pfSense Guide**: [docs.netgate.com](https://docs.netgate.com)
- **Prometheus Documentation**: [prometheus.io/docs](https://prometheus.io/docs)
- **Grafana Tutorials**: [grafana.com/tutorials](https://grafana.com/tutorials)
- **Suricata Rules**: [rules.emergingthreats.net](https://rules.emergingthreats.net)

## 📝 Notes

This home lab infrastructure is designed for learning, testing, and demonstrating enterprise network and security concepts. All configurations follow industry best practices and security standards.

---

**Author**: Gopi Pathuri  
**LinkedIn**: [linkedin.com/in/gopi-pathuri](https://linkedin.com/in/gopi-pathuri)  
**GitHub**: [github.com/gopi-pathuri](https://github.com/gopi-pathuri)
