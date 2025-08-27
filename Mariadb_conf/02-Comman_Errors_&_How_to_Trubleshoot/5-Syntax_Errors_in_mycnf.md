## Syntax Errors in my.cnf

**Error:**
MariaDB fails to start due to invalid configuration file syntax (e.g., missed semicolons, improper section headers).

**How to Simulate:**

- Misspell a section header:

```ini
[missingsection]
```

- Insert a typo in a variable:

```ini
innodb_buffer_poolz_size=1G
```

- Restart MariaDB after changes will fail.

**How to Fix:**

- Check MariaDB logs for syntax errors:

```bash
sudo journalctl -xeu mariadb
```

- Use `my_print_defaults` to validate configuration:

```bash
my_print_defaults mysqld
```

- Correct the typos and restart MariaDB:

```bash
sudo systemctl restart mariadb
```

**Lesson:**

- Typos or misplaced section headers in `my.cnf` will prevent MariaDB from starting.
- Always double-check syntax and use tools to validate config files.
