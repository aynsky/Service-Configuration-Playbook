# MariaDB Troubleshooting Notes

## 1. Incorrect File Permissions on my.cnf

**Error:**
MariaDB fails to start due to its main config file (on RHEL 9 `/etc/my.cnf.d/mariadb-server.cnf`) not being readable by the server.

**How to Simulate:**

```bash
chmod 000 /etc/mysql/my.cnf
```

- Makes the config file unreadable for any user.
- Attempt to restart MariaDB after this will fail.

**How to Fix:**

```bash
chmod 600 /etc/mysql/my.cnf
chown mysql:mysql /etc/mysql/my.cnf
```

- `600` → only owner can read/write
- Owner set to `mysql` user, ensuring MariaDB can read it.

**Lesson:**

- Always secure configuration files to prevent unauthorized access.
- Ensure MariaDB can still read the file to start correctly.

---
