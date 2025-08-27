## Incorrect bind-address

**Error:**
Remote clients can't connect; MariaDB only listens on localhost.

**How to Simulate:**

- Set `bind-address = 127.0.0.1` in `[mysqld]` section of `my.cnf`.

**How to Fix:**

- Change `bind-address` to `0.0.0.0` to allow connections from any host, or to the server's specific IP for controlled access:

```ini
[mysqld]
bind-address = 0.0.0.0
```

- Ensure the firewall allows traffic on port 3306.
- Restart MariaDB after changes:

```bash
sudo systemctl restart mariadb
```

**Lesson:**

- Properly configure `bind-address` depending on your use case:
  - Local development → `127.0.0.1`
  - Remote access → server IP or `0.0.0.0` with firewall rules.
- Always verify connectivity from remote clients after making changes.
