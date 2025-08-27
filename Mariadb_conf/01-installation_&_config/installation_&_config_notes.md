```markdown
# MariaDB Installation on Rocky Linux (or similar)

## 1️⃣ Install MariaDB Server

sudo dnf install mariadb-server -y

- `dnf` installs the MariaDB server package and required dependencies.
- `-y` automatically confirms installation.

## 2️⃣ Enable and Start Service

sudo systemctl enable --now mariadb

- `enable` → ensures MariaDB starts on boot.
- `--now` → starts the service immediately.

## 3️⃣ Verify Service Status

sudo systemctl status mariadb

- Look for `active (running)` status.
- Check for any error messages if it didn’t start.

## 4️⃣ Test Connection

mysql -u root

- By default, MariaDB uses **socket authentication** for root.
- If it connects, the server is running fine.

## 5️⃣ Secure Installation

sudo mysql_secure_installation

- Recommended settings:
  - **Set root password:** Yes (choose a strong password)
  - **Remove anonymous users:** Yes (prevents unauthorized access)
  - **Disallow root login remotely:** Yes (enhances security, root can only login locally)
  - **Remove test database and access:** Yes (test DB is public and unnecessary in production)
  - **Reload privilege tables:** Yes (applies all changes immediately)

> ✅ After this step, MariaDB is installed and secured.
```

---

## 2. Basic User & Database Operations

### File: 03_user_db_ops.md

```markdown
# MariaDB Basic User & Database Operations

## 1️⃣ Create a Test Database

CREATE DATABASE testdb;

- Creates a new database named `testdb`.

## 2️⃣ Create a User

CREATE USER 'testuser'@'%' IDENTIFIED BY 'StrongPass!';

- `%` allows connections from **any host** (useful for remote testing).
- Choose a **strong password**.

## 3️⃣ Grant Privileges

GRANT ALL PRIVILEGES ON testdb.\* TO 'testuser'@'%';
FLUSH PRIVILEGES;

- Gives `testuser` full access to `testdb`.
- `FLUSH PRIVILEGES` reloads permissions immediately.

## 4️⃣ Verify User Login

mysql -u testuser -p testdb

- Enter the password (`StrongPass!`).
- If it connects and you can see/use the database, setup is successful.

---

### 🔹 Notes / Tips

- Always check the MariaDB service status before making changes.
- Use `%` carefully in production; for local tests, it's fine.
- Keep your `my.cnf` config backed up if you make changes.
```
