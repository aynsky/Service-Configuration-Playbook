## Directory Ownership Issues

**Error:**
MariaDB fails to access its data directories.

**How to Simulate:**

- Change the `/var/lib/mysql` directory ownership to a different user:

```bash
sudo chown -R someuser:someuser /var/lib/mysql
```

- Attempt to start MariaDB will fail.

**How to Fix:**

- Revert ownership back to `mysql:mysql`:

```bash
sudo chown -R mysql:mysql /var/lib/mysql
```

- Ensure proper permissions:

```bash
sudo chmod 755 /var/lib/mysql
```

- Restart MariaDB:

```bash
sudo systemctl restart mariadb
```

**Lesson:**

- MariaDB must always run with correct user permissions.
- Incorrect ownership prevents the server from reading or writing to its data directories, leading to startup failure.
