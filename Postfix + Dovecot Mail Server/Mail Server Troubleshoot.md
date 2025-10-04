# Mail Server Troubleshooting Guide

This comprehensive guide provides tools and commands to identify and resolve mail delivery issues in both lab and production environments.

---

## Quick Diagnostic Workflow

When troubleshooting mail issues, follow this systematic approach:

1. **Check service status** → Are Postfix/Dovecot running?
2. **Review mail queue** → Are messages stuck?
3. **Examine logs** → What errors are reported?
4. **Test connectivity** → Can servers communicate?
5. **Verify DNS** → Are hostnames resolving correctly?
6. **Check firewall** → Are required ports open?
7. **Test authentication** → Can users login?

---

## 1. Mail Queue Management

### View Mail Queue Status

```bash
# Display mail queue (both commands show the same information)
mailq
postqueue -p

# Show detailed queue information
postqueue -p | more

# Count messages in queue
mailq | grep -c "^[A-Z0-9]"
```

### Manage Queue Items

```bash
# Flush mail queue (retry all messages immediately)
sudo postqueue -f

# Process specific message
sudo postqueue -i <queue_id>

# Delete specific message
sudo postsuper -d <queue_id>

# Delete all messages in queue
sudo postsuper -d ALL

# Delete all deferred messages
sudo postsuper -d ALL deferred

# Hold/release messages
sudo postsuper -h <queue_id>    # Hold
sudo postsuper -H <queue_id>    # Release
```

---

## 2. Postfix Service Management & Logs

### Service Status and Control

```bash
# Check Postfix service status
sudo systemctl status postfix

# Start/Stop/Restart Postfix
sudo systemctl start postfix
sudo systemctl stop postfix
sudo systemctl restart postfix

# Reload configuration without interrupting service
sudo postfix reload
sudo systemctl reload postfix

# Check if Postfix is running
sudo postfix status
```

### Log Analysis

```bash
# Follow live mail log (location varies by distribution)
sudo tail -f /var/log/maillog        # RHEL/CentOS
sudo tail -f /var/log/mail.log       # Ubuntu/Debian
sudo tail -f /var/log/mail.err       # Error log on Debian systems

# View last 100 lines
sudo tail -n 100 /var/log/maillog

# Search for specific email address
sudo grep "user@example.com" /var/log/maillog

# Check detailed errors in systemd journal
sudo journalctl -xeu postfix
sudo journalctl -u postfix --since "1 hour ago"
sudo journalctl -u postfix -f    # Follow live

# Find all mail-related logs
sudo find /var/log -name "*mail*" -type f
```

---

## 3. Network & Connectivity Diagnostics

### Test SMTP Connectivity

```bash
# Test connection to mail server (port 25)
telnet mail.example.com 25
nc -vz mail.example.com 25
nmap -p 25 mail.example.com

# Test with timeout
timeout 5 telnet mail.example.com 25

# Test SMTP manually
telnet mail.example.com 25
# Then type:
EHLO testclient
MAIL FROM: <test@example.com>
RCPT TO: <user@example.com>
DATA
Subject: Test
.
QUIT
```

### Check Listening Ports

```bash
# Check which IP/port Postfix is listening on
sudo ss -tlnp | grep :25
sudo netstat -tlnp | grep :25
sudo lsof -i :25

# Check all mail-related ports
sudo ss -tlnp | grep -E ':25|:143|:110|:993|:995'

# Verify Postfix master process
ps aux | grep -E 'master|postfix'
```

### Network Interface and Routing

```bash
# Show all network interfaces
ip addr show
ifconfig -a

# Check routing table
ip route show
route -n

# Test network path to mail server
traceroute mail.example.com
mtr mail.example.com

# Ping mail server
ping -c 4 10.0.1.10

# Check for packet loss
ping -c 100 -i 0.2 mail.example.com | grep loss
```

---

## 4. Firewall Troubleshooting

### Firewall Status (firewalld - RHEL/CentOS)

```bash
# List all firewall rules
sudo firewall-cmd --list-all

# Check if SMTP service/port is allowed
sudo firewall-cmd --list-services
sudo firewall-cmd --list-ports

# Add SMTP port
sudo firewall-cmd --add-port=25/tcp --permanent
sudo firewall-cmd --add-service=smtp --permanent
sudo firewall-cmd --reload

# Add mail-related ports
sudo firewall-cmd --add-port=25/tcp --permanent    # SMTP
sudo firewall-cmd --add-port=143/tcp --permanent   # IMAP
sudo firewall-cmd --add-port=110/tcp --permanent   # POP3
sudo firewall-cmd --add-port=993/tcp --permanent   # IMAPS
sudo firewall-cmd --add-port=995/tcp --permanent   # POP3S
sudo firewall-cmd --reload
```

### Firewall Status (ufw - Ubuntu/Debian)

```bash
# Check status
sudo ufw status verbose

# Allow mail ports
sudo ufw allow 25/tcp
sudo ufw allow 143/tcp
sudo ufw allow 110/tcp

# Check iptables directly
sudo iptables -L -n -v
```

---

## 5. DNS & Hostname Resolution

### DNS Lookup Tests

```bash
# Multiple ways to check DNS resolution
host mail.example.com
nslookup mail.example.com
dig mail.example.com
dig +short mail.example.com

# Check MX records
dig MX example.com
host -t MX example.com
nslookup -query=mx example.com

# Check reverse DNS
dig -x 10.0.1.10
host 10.0.1.10

# Check DNS server being used
cat /etc/resolv.conf
resolvectl status    # SystemD systems
```

### Local Hostname Override

```bash
# Edit hosts file for local resolution
sudo vi /etc/hosts
# Add entry:
10.0.1.10 mail.example.com mail

# Verify hosts file is being used
getent hosts mail.example.com

# Check hostname configuration
hostname
hostname -f
hostnamectl
```

---

## 6. Dovecot Authentication & Mailbox Checks

### Authentication Testing

```bash
# Test user authentication
sudo doveadm auth test username
sudo doveadm auth test username password

# Test authentication cache
sudo doveadm auth cache flush

# List all users
sudo doveadm user '*'

# Check user details
sudo doveadm user username
```

### Mailbox Inspection

```bash
# Check mailbox content
sudo doveadm fetch -u username text mailbox INBOX
sudo doveadm fetch -u username hdr mailbox INBOX

# List mailbox folders
sudo doveadm mailbox list -u username

# Check mailbox status
sudo doveadm mailbox status -u username messages INBOX

# Verify Maildir structure
sudo ls -la /home/username/Maildir/
sudo ls -la /home/username/Maildir/new/
sudo ls -la /home/username/Maildir/cur/

# Check mailbox permissions
sudo stat /home/username/Maildir/
```

### Dovecot Service Management

```bash
# Check Dovecot status
sudo systemctl status dovecot

# Restart Dovecot
sudo systemctl restart dovecot

# Check Dovecot configuration
sudo doveconf -n
sudo doveconf -n | grep mail_location

# Test Dovecot protocols
sudo doveadm log test
```

---

## 7. Postfix Configuration Verification

### Configuration Analysis

```bash
# List all Postfix configuration
postconf

# Show only non-default values
postconf -n

# Check specific parameters
postconf myhostname
postconf mydomain
postconf mynetworks
postconf relayhost
postconf mydestination

# Verify mail delivery restrictions
postconf smtpd_recipient_restrictions
postconf smtpd_relay_restrictions
postconf smtpd_client_restrictions

# Check relay configuration
postconf relayhost
postconf transport_maps
postconf relay_domains

# Validate configuration syntax
sudo postfix check

# Show mail routing
postmap -q user@example.com hash:/etc/postfix/transport
```

---

## 8. Common Issues and Solutions

### Issue Resolution Matrix

| **Symptom** | **Diagnostic Commands** | **Common Solutions** |
|-------------|------------------------|---------------------|
| **Unknown user / delivery failure** | `doveadm auth test username`<br>`ls -la /home/username/Maildir` | • Create user account<br>• Initialize Maildir<br>• Fix permissions |
| **Connection refused to mail server** | `telnet mail.example.com 25`<br>`ss -tlnp \| grep :25` | • Start Postfix service<br>• Check `inet_interfaces`<br>• Fix firewall rules |
| **Mail queue stuck / not delivering** | `mailq`<br>`tail -f /var/log/maillog` | • Check `relayhost` setting<br>• Verify DNS resolution<br>• Flush queue with `postqueue -f` |
| **Firewall blocking SMTP** | `firewall-cmd --list-all`<br>`nc -vz <IP> 25` | • Add port 25/tcp<br>• Reload firewall<br>• Check iptables rules |
| **Wrong IP / multi-NIC routing** | `ip addr show`<br>`ip route show` | • Check `/etc/hosts`<br>• Verify routing table<br>• Set correct `inet_interfaces` |
| **DNS resolution issues** | `host mail.example.com`<br>`dig mail.example.com` | • Check `/etc/resolv.conf`<br>• Add to `/etc/hosts`<br>• Verify DNS server |
| **Authentication failures** | `doveadm auth test user pass`<br>`journalctl -u dovecot` | • Reset password<br>• Check PAM configuration<br>• Verify `auth_mechanisms` |
| **Permission denied** | `ls -la /var/spool/postfix`<br>`stat /home/user/Maildir` | • Fix ownership with `chown`<br>• Correct permissions (700 for Maildir) |

---

## 9. Advanced Debugging

### Enable Verbose Logging

```bash
# Enable debug logging in Postfix
sudo postconf -e "debug_peer_list = 127.0.0.1"
sudo postconf -e "debug_peer_level = 3"
sudo postfix reload

# Enable verbose logging for specific domains
sudo postconf -e "debug_peer_list = example.com"

# Enable Dovecot debug logging
# Edit /etc/dovecot/conf.d/10-logging.conf
auth_debug = yes
mail_debug = yes
verbose_ssl = yes
```

### Performance Analysis

```bash
# Check mail processing rate
sudo pflogsumm /var/log/maillog

# Monitor real-time mail flow
sudo tail -f /var/log/maillog | grep -E "from=|to=|status="

# Check system resources
top -p $(pgrep -d',' postfix)
iostat -x 1
vmstat 1
```

### Testing Mail Delivery Path

```bash
# Trace message delivery
sudo postqueue -v -f    # Verbose queue flush

# Send test with verbose output
echo "Test" | mail -v -s "Subject" user@example.com

# Check specific message in queue
sudo postcat -q <queue_id>
```

---

## 10. Emergency Recovery

### When Everything Fails

```bash
# 1. Stop all services
sudo systemctl stop postfix dovecot

# 2. Backup current configuration
sudo cp -r /etc/postfix /etc/postfix.backup.$(date +%Y%m%d)
sudo cp -r /etc/dovecot /etc/dovecot.backup.$(date +%Y%m%d)

# 3. Check for configuration errors
sudo postfix check
sudo doveconf -n > /tmp/dovecot-check.txt

# 4. Reset to minimal configuration
sudo postconf -e "inet_interfaces = loopback-only"
sudo postconf -e "mydestination = localhost"

# 5. Start services one by one
sudo systemctl start postfix
sudo systemctl start dovecot

# 6. Gradually restore configuration
```

### Log Collection for Support

```bash
# Collect all relevant logs
sudo tar czf mail-debug-$(date +%Y%m%d).tar.gz \
  /var/log/mail* \
  /etc/postfix/ \
  /etc/dovecot/ \
  /var/spool/postfix/defer/ \
  /var/spool/postfix/deferred/

# System information
uname -a > system-info.txt
postconf -n >> system-info.txt
doveconf -n >> system-info.txt
ss -tlnp >> system-info.txt
```

---

## Quick Reference Card

### Most Used Commands

```bash
# Service Control
systemctl status postfix dovecot
systemctl restart postfix dovecot

# Queue Management  
mailq                      # View queue
postqueue -f               # Flush queue
postsuper -d ALL           # Clear queue

# Testing
telnet localhost 25        # Test SMTP
doveadm auth test user     # Test auth
tail -f /var/log/maillog   # Watch logs

# Configuration
postconf -n                # Show config
postfix check              # Validate
doveconf -n                # Dovecot config
```

### Critical File Locations

- **Postfix Config**: `/etc/postfix/main.cf`
- **Dovecot Config**: `/etc/dovecot/dovecot.conf`
- **Mail Logs**: `/var/log/maillog` or `/var/log/mail.log`
- **Mail Queue**: `/var/spool/postfix/`
- **User Mailboxes**: `/home/*/Maildir/`

---

## Notes

- Always check logs first - they usually contain the exact error
- Most issues are either DNS, firewall, or permission related
- Test incrementally - verify each component works before moving to the next
- Keep configuration backups before making changes
- Document any custom configurations for future reference