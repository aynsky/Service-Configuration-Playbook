# Postfix + Dovecot Mail Server Configuration Guide

## Overview

This guide walks through setting up a complete email infrastructure with Postfix as the Mail Transfer Agent (MTA) and Dovecot as the IMAP/POP3 server. The configuration includes a main mail server that handles email storage and retrieval, plus an application server that relays mail through the main server without storing mail locally.

### What This Setup Accomplishes

- **Mail Server**: Receives and stores email for your domain, provides IMAP/POP3 access
- **Application Server**: Sends system-generated emails (alerts, notifications) through the mail server without local mail storage
- **Internal Network**: Secure relay configuration allowing trusted servers to send mail

## Prerequisites

### Lab Environment

| Component      | Hostname Example          | IP Example    | Role                          |
|----------------|--------------------------|---------------|-------------------------------|
| Mail Server    | mail.example.domain      | 10.0.1.10     | Postfix + Dovecot (MTA + IMAP) |
| App Server     | app01.example.domain     | 10.0.1.20     | Relay client (sends mail)     |

### System Requirements

- Operating System: RHEL/CentOS 8+ or Ubuntu 20.04+
- Network: Internal network with static IP addresses
- DNS: Proper hostname resolution between servers
- Firewall: Ability to configure ports

---

## Installation

### On Mail Server

#### RHEL/CentOS
```bash
sudo dnf install -y postfix dovecot
sudo systemctl enable postfix dovecot
```

#### Ubuntu/Debian
```bash
sudo apt update
sudo apt install -y postfix dovecot-core dovecot-imapd dovecot-pop3d
sudo systemctl enable postfix dovecot
```

### On Application Server

#### RHEL/CentOS
```bash
sudo dnf install -y postfix s-nail
sudo systemctl enable postfix
```

#### Ubuntu/Debian
```bash
sudo apt update
sudo apt install -y postfix s-nail
sudo systemctl enable postfix
```

> **Note**: We use `s-nail` instead of traditional `mailx` as it provides better MIME support, improved security features, and is actively maintained as the modern replacement for mailx.

---

## 1. Mail Server Configuration

### 1.1 Postfix Configuration (`/etc/postfix/main.cf`)

Edit the main Postfix configuration file:

```ini
# Server identity
myhostname = mail.example.domain
mydomain = example.domain
myorigin = $mydomain

# Network interfaces and protocols
inet_interfaces = all           # Listen on all network interfaces
inet_protocols = ipv4           # Use IPv4 only (or 'all' for IPv4+IPv6)

# Local delivery destinations
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

# Mailbox format and location
home_mailbox = Maildir/         # Use Maildir format (one file per message)

# Trusted networks allowed to relay
mynetworks = 127.0.0.0/8 10.0.1.0/24    # Adjust subnet to match your network

# Access control
smtpd_recipient_restrictions = 
    permit_mynetworks,
    permit_sasl_authenticated,
    reject_unauth_destination
```

**Key Configuration Notes:**
- `inet_interfaces = all` enables Postfix to accept connections from external hosts
- `mynetworks` must include your internal subnet to allow relay from trusted servers
- `home_mailbox = Maildir/` uses the more reliable Maildir format over mbox

### 1.2 Dovecot Configuration

#### Enable IMAP and POP3 Services

Edit `/etc/dovecot/dovecot.conf`:

```ini
# Protocols to enable
protocols = imap pop3 lmtp

# Listen on all interfaces
listen = *
```

#### Configure Mail Location

Edit `/etc/dovecot/conf.d/10-mail.conf`:

```ini
# Mail storage location
mail_location = maildir:~/Maildir

# Required for Postfix integration
mail_privileged_group = mail
```

#### Authentication Settings

Edit `/etc/dovecot/conf.d/10-auth.conf`:

```ini
# Enable plain text authentication (use with SSL in production)
disable_plaintext_auth = no
auth_mechanisms = plain login
```

### 1.3 User Account Setup

Create mail users and initialize their mailboxes:

```bash
# Add a mail user (if not existing)
sudo useradd -m -s /bin/bash mailuser1

# Set password
sudo passwd mailuser1

# Create Maildir structure
sudo -u mailuser1 mkdir -p /home/mailuser1/Maildir/{new,cur,tmp}

# Set proper permissions
sudo chown -R mailuser1:mailuser1 /home/mailuser1/Maildir
sudo chmod -R 700 /home/mailuser1/Maildir
```

### 1.4 Service Management

```bash
# Restart services
sudo systemctl restart postfix dovecot

# Verify services are running
sudo systemctl status postfix dovecot
```

### 1.5 Firewall Configuration

```bash
# Open required ports
sudo firewall-cmd --add-service=smtp --permanent      # Port 25
sudo firewall-cmd --add-service=imap --permanent      # Port 143
sudo firewall-cmd --add-service=pop3 --permanent      # Port 110
sudo firewall-cmd --reload

# For Ubuntu/Debian with ufw
sudo ufw allow 25/tcp
sudo ufw allow 143/tcp
sudo ufw allow 110/tcp
```

### 1.6 Testing Mail Server

Test Dovecot authentication:

```bash
# Test user authentication
sudo doveadm auth test mailuser1

# Check mailbox contents
sudo doveadm fetch -u mailuser1 text mailbox INBOX
```

---

## 2. Application Server Configuration (Relay Client)

### 2.1 Postfix Relay Configuration (`/etc/postfix/main.cf`)

Configure the application server to relay all mail through the main mail server:

```ini
# Server identity
myhostname = app01.example.domain
mydomain = example.domain
myorigin = $mydomain         # Append domain to unqualified addresses

# Relay configuration
relayhost = [10.0.1.10]:25   # IP of mail server in brackets to skip MX lookup

# Disable local mail delivery
mydestination =              # Empty: no local delivery

# Network configuration  
inet_interfaces = loopback-only    # Only listen on localhost
inet_protocols = ipv4              # IPv4 only

# Security settings
mynetworks = 127.0.0.0/8          # Only trust localhost
```

**Configuration Explained:**
- `relayhost`: All outbound mail is forwarded to this server
- `mydestination = ` (empty): Prevents local mail delivery
- `inet_interfaces = loopback-only`: Prevents external SMTP connections
- Brackets around IP in `relayhost` skip DNS MX record lookup

### 2.2 Service Management

```bash
# Restart Postfix
sudo systemctl restart postfix

# Verify configuration
sudo postconf -n
```

### 2.3 Testing Relay Functionality

Send a test email using s-nail:

```bash
# Send test email
echo "Test email body from application server" | s-nail -s "Test Relay Subject" user@example.domain

# Check mail queue
mailq

# View mail log
sudo tail -f /var/log/maillog    # RHEL/CentOS
sudo tail -f /var/log/mail.log   # Ubuntu/Debian
```

---

## Verification Steps

### On Mail Server

1. Check if mail was received:
```bash
# Check Postfix queue
sudo mailq

# Check mail logs for delivery
sudo grep "to=<user@example.domain>" /var/log/maillog

# Verify mail in user's Maildir
sudo ls -la /home/mailuser1/Maildir/new/
```

2. Test IMAP access:
```bash
# Connect to IMAP (basic test)
telnet localhost 143
# Type: a login mailuser1 password
# Type: b select inbox
# Type: c logout
```

### On Application Server

1. Verify relay configuration:
```bash
# Check that mail is being relayed (not delivered locally)
sudo postconf mydestination relayhost
```

2. Monitor sending:
```bash
# Watch mail queue
watch -n 2 'mailq'
```

---

## Security Considerations

1. **Production Environment**:
   - Enable SASL authentication for relay access
   - Implement TLS/SSL for encrypted connections
   - Use strong passwords and consider key-based authentication
   - Restrict `mynetworks` to specific trusted IPs only

2. **Monitoring**:
   - Regularly check mail logs for unauthorized relay attempts
   - Monitor queue sizes to detect mail floods
   - Set up fail2ban to block repeated authentication failures

3. **Backup**:
   - Regular backup of `/home/*/Maildir` directories
   - Backup Postfix and Dovecot configuration files
   - Document any custom configurations

---

## Next Steps

- Configure SSL/TLS for secure connections
- Set up DKIM, SPF, and DMARC for email authentication
- Implement spam filtering with SpamAssassin or rspamd
- Configure webmail access with Roundcube or similar
- Set up automated monitoring and alerting

---

## Quick Reference

### Common Commands

```bash
# Postfix
sudo postfix check              # Check configuration
sudo postconf -n                # Show non-default settings
sudo postqueue -p               # Show mail queue
sudo postsuper -d ALL           # Delete all queued mail

# Dovecot
sudo doveadm user '*'           # List all users
sudo doveadm quota get -u user  # Check user quota
sudo doveconf -n                # Show configuration

# Testing
echo "test" | mail -s "subject" user@domain  # Send test mail
sudo tail -f /var/log/mail*    # Monitor mail logs
```

### Configuration File Locations

- Postfix main config: `/etc/postfix/main.cf`
- Postfix master config: `/etc/postfix/master.cf`
- Dovecot main config: `/etc/dovecot/dovecot.conf`
- Dovecot config directory: `/etc/dovecot/conf.d/`
- Mail spool: `/var/spool/mail/` or `/home/*/Maildir/`