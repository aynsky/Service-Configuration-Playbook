## Client-Server Version Mismatch

**Error:**
Cannot connect from client; odd authentication errors may occur.

**How to Simulate:**

- Use an older or incompatible MariaDB/MySQL client to connect to the server.

**How to Fix:**

- Ensure the client version matches the server's supported version.
- Check versions:

```bash
mysql --version  # client version
SELECT VERSION();  # server version
```

- Update the client or server to compatible versions if necessary.

**Lesson:**

- Always ensure client libraries are compatible with the server version.
- Mismatched versions can cause unexpected authentication or connection errors.
