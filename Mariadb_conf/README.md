# mariadb-lab# MariaDB Lab Notes

Welcome to the MariaDB Lab repository! This repository is designed to help you **learn, configure, and troubleshoot MariaDB** in a structured, hands-on way. The exercises include installations, basic configuration, and common real-world errors to practice.

## Structure

- **01-installation/**: Notes on installing MariaDB and initial setup.
- **02-Comman*Errors*&\_How_to_Trubleshoot/**: Individual error scenarios with simulation, fix, and lessons learned.

## Quick Troubleshooting Flow

If MariaDB/MySQL refuses to start:

1. **Check logs first:**

```bash
journalctl -xeu mariadb
```

2. **If logs are unclear, start manually in safe mode:**

```bash
mysqld_safe --user=mysql
```

- This bypasses systemd and runs MariaDB in a “safe mode,” showing errors directly in the terminal.

### Recommended Workflow

- Check service status:

```bash
systemctl status mariadb
```

- Check logs:

```bash
journalctl -xeu mariadb
```

- If still unclear → start with:

```bash
mysqld_safe --user=mysql
```

## Safety Notes

- Each simulated mistake is **safe to perform on non-production VMs or local setups**.
- Always **back up your configuration** before making changes.
- Check MariaDB logs after each simulated problem for a deeper understanding.

This repository is intended as a **hands-on guide** to make you confident in MariaDB administration, configuration, and troubleshooting.
