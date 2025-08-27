## Firewall or Port Misconfiguration

**Error:**
Cannot connect to MariaDB server from another host.

**How to Simulate:**

- Block port 3306 with firewall rules:

```bash
sudo firewall-cmd --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
```

- Then simulate blocking by removing the rule or using `--remove-port`.

**How to Fix:**

- Open port 3306 in the firewall:

```bash
sudo firewall-cmd --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
```

- Confirm port is listening:

```bash
sudo ss -tuln | grep 3306
```

**Lesson:**

- Always verify infrastructure settings (firewalls, network rules) alongside MariaDB configuration.
- Remote access requires correct bind-address and firewall port openings.
