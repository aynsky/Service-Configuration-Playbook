## Socket Mismatch

**Error:**
Local client cannot connect to MariaDB; "can't connect to local MySQL server through socket".

**How to Simulate:**

- Set different socket paths for `[client]` and `[mysqld]` sections in `my.cnf`.
- Or change the permission on `/var/lib/mysql` to restrict access.

**How to Fix:**

- Ensure both `[client]` and `[mysqld]` sections point to the same socket file:

```ini
[mysqld]
socket=/var/lib/mysql/mysql.sock

[client]
socket=/var/lib/mysql/mysql.sock
```

- If permissions were changed, restore correct ownership and read/write permissions for MySQL:

```bash
sudo chown -R mysql:mysql /var/lib/mysql
sudo chmod 755 /var/lib/mysql
```

- Restart MariaDB after changes:

```bash
sudo systemctl restart mariadb
```

**Lesson:**

- Always check the `socket` entry under both `[client]` and `[mysqld]` sections.
- Permissions on the data directory are crucial; incorrect permissions will prevent connections even if the paths match.
- Restart the service after making corrections to apply changes.
