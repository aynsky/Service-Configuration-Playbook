## Max Connections Limit (max_connections)

**Error:**
New connections can't be established; "Too many connections".

**How to Simulate:**

- Set `max_connections` very low in `my.cnf`:

```ini
[mysqld]
max_connections = 5
```

- Open multiple client connections until the limit is reached.

**How to Fix:**

- Increase `max_connections` to a value appropriate for your expected workload:

```ini
[mysqld]
max_connections = 200
```

- Restart MariaDB to apply changes:

```bash
sudo systemctl restart mariadb
```

**Lesson:**

- Always estimate anticipated user/application connections.
- Setting this value too low blocks legitimate connections; too high may exhaust server resources.
