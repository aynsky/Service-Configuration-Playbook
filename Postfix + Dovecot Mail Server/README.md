# Mail Server Infrastructure Documentation

## 📚 Overview

This documentation provides a complete guide for deploying and maintaining a Postfix + Dovecot mail server infrastructure. It covers everything from initial setup to advanced troubleshooting, designed for both lab environments and production deployments.

### Documentation Structure

1. **[Configuration Guide](./mail-server-configuration.md)** - Complete setup and deployment instructions
2. **[Troubleshooting Guide](./mail-server-troubleshooting.md)** - Diagnostic tools and problem resolution

---

## 🎯 Project Goals

This mail infrastructure setup achieves:

- **Centralized Mail Server**: Full-featured mail server with SMTP, IMAP, and POP3 support
- **Relay Configuration**: Application servers that send mail through the central server
- **Scalable Architecture**: Easy to add additional relay clients
- **Security**: Network-based access control and authentication
- **Reliability**: Maildir format for robust mail storage

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                   Internal Network                      │
│                     (10.0.1.0/24)                      │
│                                                         │
│  ┌──────────────────┐         ┌──────────────────┐    │
│  │  App Server 01   │ ──25──> │   Mail Server    │    │
│  │  (Relay Client)  │         │  (MTA + IMAP)    │    │
│  │   10.0.1.20     │         │    10.0.1.10     │    │
│  └──────────────────┘         └──────────────────┘    │
│                                    │                    │
│  ┌──────────────────┐              │                   │
│  │  App Server 02   │ ──25────────┘                   │
│  │  (Relay Client)  │                                  │
│  │   10.0.1.21     │         ┌──────────────────┐    │
│  └──────────────────┘         │   Mail Clients   │    │
│                               │  (IMAP/POP3)     │    │
│                               └──────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

### Component Roles

| Component | Purpose | Services |
|-----------|---------|----------|
| **Mail Server** | Central mail handling | Postfix (SMTP), Dovecot (IMAP/POP3) |
| **App Servers** | Application mail relay | Postfix (relay-only mode) |
| **Mail Clients** | End-user access | Thunderbird, Outlook, Webmail |

---

## 📋 Prerequisites

### System Requirements

- **Operating System**: 
  - RHEL/CentOS 8+ or Ubuntu 20.04+
  - Minimum 2GB RAM for mail server
  - 20GB+ disk space for mail storage

- **Network Requirements**:
  - Static IP addresses for all servers
  - Internal DNS or `/etc/hosts` configuration
  - Firewall management access

- **Knowledge Requirements**:
  - Basic Linux system administration
  - Understanding of SMTP protocol
  - Network troubleshooting skills

---

## 🚀 Quick Start Guide

### Step 1: Initial Setup
Follow the [Configuration Guide](./mail-server-configuration.md) to:
1. Install Postfix and Dovecot packages
2. Configure the mail server
3. Set up application server relay
4. Create user mailboxes
5. Configure firewall rules

### Step 2: Verification
Use the [Troubleshooting Guide](./mail-server-troubleshooting.md) to:
1. Verify service status
2. Test mail delivery
3. Check authentication
4. Monitor mail flow

### Step 3: Testing Commands

```bash
# On Mail Server - Quick Health Check
systemctl status postfix dovecot
mailq
doveadm user '*'

# On App Server - Send Test Mail
echo "Test mail" | s-nail -s "Test" user@example.domain

# Verify Delivery
doveadm fetch -u user text mailbox INBOX
```

---

## 📖 Documentation Guide

### [1. Configuration Guide](./mail-server-configuration.md)

Complete setup instructions including:

- **Installation**
  - Package installation for different distributions
  - Initial service configuration

- **Mail Server Setup**
  - Postfix main.cf configuration
  - Dovecot IMAP/POP3 setup
  - User account creation
  - Maildir initialization

- **Application Server Setup**
  - Relay-only Postfix configuration
  - Network trust relationships

- **Security Hardening**
  - Firewall configuration
  - Access control lists
  - Authentication setup

### [2. Troubleshooting Guide](./mail-server-troubleshooting.md)

Comprehensive diagnostic reference including:

- **Quick Diagnostic Workflow**
  - Systematic troubleshooting approach
  - Common issue resolution

- **Service Management**
  - Queue management
  - Log analysis
  - Service control

- **Network Diagnostics**
  - Connectivity testing
  - DNS resolution
  - Firewall verification

- **Advanced Debugging**
  - Performance analysis
  - Verbose logging
  - Emergency recovery

---

## 🔧 Common Use Cases

### Lab Environment Setup
Perfect for:
- Learning mail server administration
- Testing mail-dependent applications
- Development environment setup

### Production Deployment
Suitable for:
- Small to medium business email
- Application mail relay infrastructure
- Internal notification systems

### Hybrid Configuration
Can be extended for:
- Multi-site deployments
- High availability setup
- Integration with external mail services

---

## 🛠️ Maintenance Tasks

### Daily Operations
```bash
# Check mail queue
mailq

# Monitor logs
tail -f /var/log/maillog

# Check disk usage
df -h /home
```

### Weekly Tasks
```bash
# Review mail statistics
pflogsumm /var/log/maillog

# Check for failed deliveries
grep "status=deferred" /var/log/maillog | tail -20

# Verify user authentication
doveadm auth test username
```

### Monthly Tasks
```bash
# Backup configuration
tar czf mail-config-$(date +%Y%m).tar.gz /etc/postfix /etc/dovecot

# Review and clean old logs
find /var/log -name "maillog-*" -mtime +30 -delete

# Update systems
yum update postfix dovecot  # or apt update && apt upgrade
```

---

## 🚨 Troubleshooting Quick Reference

### Most Common Issues

| Issue | Quick Fix | Details |
|-------|-----------|---------|
| Mail stuck in queue | `postqueue -f` | See [Troubleshooting Guide §1](./mail-server-troubleshooting.md#1-mail-queue-management) |
| Connection refused | Check firewall: `firewall-cmd --list-all` | See [Troubleshooting Guide §4](./mail-server-troubleshooting.md#4-firewall-troubleshooting) |
| Unknown user | `doveadm auth test user` | See [Troubleshooting Guide §6](./mail-server-troubleshooting.md#6-dovecot-authentication--mailbox-checks) |
| DNS issues | Add to `/etc/hosts` | See [Troubleshooting Guide §5](./mail-server-troubleshooting.md#5-dns--hostname-resolution) |

### Emergency Contacts
Document your organization's specific contacts:
- System Administrator: [Contact]
- Network Team: [Contact]
- Emergency Escalation: [Process]

---

## 📊 Monitoring & Metrics

### Key Performance Indicators
- Queue size: Should be < 100 messages
- Delivery rate: Monitor for sudden drops
- Authentication failures: Watch for brute force attempts
- Disk usage: Mail storage growth rate

### Recommended Monitoring Tools
- **Nagios/Zabbix**: Service monitoring
- **Grafana**: Mail flow visualization
- **fail2ban**: Security monitoring
- **logwatch**: Daily log summaries

---

## 🔐 Security Considerations

### Essential Security Measures
1. **Authentication**
   - Strong passwords
   - Consider implementing SASL
   - Regular password rotation

2. **Network Security**
   - Restrict `mynetworks` to trusted IPs
   - Implement rate limiting
   - Use TLS/SSL for client connections

3. **System Hardening**
   - Regular updates
   - SELinux/AppArmor enabled
   - Minimal service exposure

### Future Security Enhancements
- [ ] Implement SPF records
- [ ] Configure DKIM signing
- [ ] Set up DMARC policies
- [ ] Deploy spam filtering (SpamAssassin/rspamd)
- [ ] Enable TLS encryption

---

## 📚 Additional Resources

### Official Documentation
- [Postfix Documentation](http://www.postfix.org/documentation.html)
- [Dovecot Wiki](https://wiki.dovecot.org/)
- [Red Hat Mail Server Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/deploying_different_types_of_servers/assembly_mail-transport-agent_deploying-different-types-of-servers)

### Community Resources
- [Mail Server Security Best Practices](https://www.ietf.org/rfc/rfc5321.txt)
- [Postfix Mailing List](http://www.postfix.org/lists.html)
- [ServerFault Mail Questions](https://serverfault.com/questions/tagged/postfix)

### Related Tools
- **Webmail**: Roundcube, SquirrelMail
- **Spam Filtering**: SpamAssassin, rspamd
- **Mail Authentication**: OpenDKIM, OpenDMARC
- **Monitoring**: Mailgraph, pflogsumm

---

## 🤝 Contributing

### Reporting Issues
When reporting issues, please include:
1. Operating system and version
2. Postfix/Dovecot versions (`postconf mail_version`, `dovecot --version`)
3. Relevant configuration files
4. Error messages from logs
5. Steps to reproduce

### Improvement Suggestions
We welcome suggestions for:
- Additional troubleshooting scenarios
- Configuration optimizations
- Security enhancements
- Documentation clarifications

---

## 📝 Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Initial | Base configuration and troubleshooting guides |
| 1.1 | TBD | Add TLS/SSL configuration |
| 1.2 | TBD | Include spam filtering setup |
| 2.0 | Future | High availability configuration |

---

## 📄 License

This documentation is provided as-is for educational and reference purposes. Always test configurations in a non-production environment first.

---

## ✅ Checklist for New Deployments

### Pre-Deployment
- [ ] Network design documented
- [ ] IP addresses assigned
- [ ] DNS entries configured
- [ ] Firewall rules planned
- [ ] User accounts listed

### Deployment
- [ ] Operating system updated
- [ ] Packages installed
- [ ] Configuration files created
- [ ] Services started
- [ ] Firewall rules applied

### Post-Deployment
- [ ] Mail delivery tested
- [ ] Authentication verified
- [ ] Monitoring configured
- [ ] Backup scheduled
- [ ] Documentation updated

---

## 🆘 Support

For additional help:
1. Review the [Configuration Guide](./mail-server-configuration.md)
2. Check the [Troubleshooting Guide](./mail-server-troubleshooting.md)
3. Search existing issues
4. Contact system administrator

---

*Last Updated: [Date]*  
*Documentation Version: 1.0*
