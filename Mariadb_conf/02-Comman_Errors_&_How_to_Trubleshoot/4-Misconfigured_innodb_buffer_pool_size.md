## Misconfigured innodb_buffer_pool_size

**Error:**
Out-of-memory errors, server crashes, or heavy swapping.

**How to Simulate:**

- Set the buffer pool much larger than available RAM. For example:

```ini
[mysqld]
innodb_buffer_pool_size = 16G
```

- On a VM with only 4GB RAM, this will likely crash MariaDB or cause heavy swapping.

**How to Fix:**

- Set `innodb_buffer_pool_size` to around 60-70% of available RAM for a dedicated database server:

```ini
[mysqld]
innodb_buffer_pool_size = 2.5G
```

- On shared systems, use even lower values to leave RAM for other processes.
- Restart MariaDB after changes:

```bash
sudo systemctl restart mariadb
```

**Lesson:**

- Always estimate RAM usage before configuring InnoDB buffer pool.
- Oversizing can degrade performance or crash the server, undersizing may cause slow queries.
