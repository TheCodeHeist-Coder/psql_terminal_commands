# PostgreSQL Cheatsheet (VPS with sudo)

## 🔌 Entering psql Shell

```bash
# Enter psql shell (default)
sudo -u postgres psql

# Enter psql shell for specific db
sudo -u postgres psql -d your_db_name
```

---

## 📋 Inside psql Shell

```sql
-- Navigation
\l              -- list all databases
\c db_name      -- switch to a database
\dt             -- list all tables in current db
\dt schema.*    -- list tables in specific schema
\d table_name   -- describe table (columns, types, indexes)
\du             -- list all users/roles
\dn             -- list all schemas
\df             -- list all functions
\dv             -- list all views
\di             -- list all indexes
\ds             -- list all sequences
\dx             -- list installed extensions

-- Get info
\conninfo       -- show current connection info
\timing         -- toggle query timing on/off
\x              -- toggle expanded display (easier to read)

-- Help
\?              -- help for psql commands
\h              -- help for SQL commands
\h CREATE TABLE -- help for specific SQL command

-- Quit
\q              -- exit psql shell
```

---

## 📊 Useful SQL Inside Shell

```sql
-- Row count of a table
SELECT COUNT(*) FROM table_name;

-- Check all table sizes
SELECT tablename, pg_size_pretty(pg_total_relation_size(tablename::text)) 
FROM pg_tables 
WHERE schemaname = 'public';

-- Check current DB size
SELECT pg_size_pretty(pg_database_size(current_database()));

-- List all tables with schema
SELECT schemaname, tablename FROM pg_tables WHERE schemaname = 'public';

-- Check active connections
SELECT pid, usename, datname, state FROM pg_stat_activity;

-- Kill a connection
SELECT pg_terminate_backend(pid);
```

---

## 📋 One-liner Commands (without entering shell)

```bash
# List all databases
sudo -u postgres psql -c "\l"

# List tables in a db
sudo -u postgres psql -d db_name -c "\dt"

# Run a query
sudo -u postgres psql -d db_name -c "SELECT COUNT(*) FROM table_name;"

# Run SQL from a file
sudo -u postgres psql -d db_name -f /tmp/script.sql
```

---

## 📦 Backup & Restore

```bash
# Dump a database (plain SQL)
sudo -u postgres pg_dump -d db_name > /tmp/db_name.sql

# Dump in compressed format
sudo -u postgres pg_dump -F c -d db_name -f /tmp/db_name.dump

# Restore from SQL file
sudo -u postgres psql -d db_name < /tmp/db_name.sql

# Restore from compressed dump
sudo -u postgres pg_restore -d db_name /tmp/db_name.dump

# Copy database directly (no temp file)
sudo -u postgres pg_dump -d source_db | sudo -u postgres psql -d target_db
```

---

## 📋 Database Operations

```bash
# Create database
sudo -u postgres psql -c "CREATE DATABASE db_name;"

# Drop database
sudo -u postgres psql -c "DROP DATABASE db_name;"

# Rename database
sudo -u postgres psql -c "ALTER DATABASE old_name RENAME TO new_name;"

# Check all DB sizes
sudo -u postgres psql -c "SELECT datname, pg_size_pretty(pg_database_size(datname)) FROM pg_database;"
```

---

## 👤 User & Permissions

```bash
# List all users
sudo -u postgres psql -c "\du"

# Create user
sudo -u postgres psql -c "CREATE USER username WITH PASSWORD 'password';"

# Grant all privileges on a database
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE db_name TO username;"

# Change user password
sudo -u postgres psql -c "ALTER USER username WITH PASSWORD 'newpassword';"

# Drop user
sudo -u postgres psql -c "DROP USER username;"
```

---

## ⚙️ Service Management

```bash
# Check postgres status
sudo systemctl status postgresql

# Start postgres
sudo systemctl start postgresql

# Stop postgres
sudo systemctl stop postgresql

# Restart postgres
sudo systemctl restart postgresql
```

---

## 🔑 Quick Tips

| Shortcut       | Action                              |
|----------------|-------------------------------------|
| `\q`           | Exit psql shell                     |
| `Ctrl + D`     | Exit psql shell                     |
| `Ctrl + C`     | Cancel current query                |
| `↑ / ↓`        | Navigate command history            |
| `\x`           | Toggle expanded view (wide tables)  |
| `;`            | Always end SQL queries with this    |

---

## 🗂️ My Dev Databases (VPS)

| Production   | Dev Copy       |
|--------------|----------------|
| postgres     | postgres_dev   |
| meta_db      | meta_db_dev    |
| shared_db    | shared_db_dev  |

### Dev Connection Strings

```dotenv
DB_URL=postgresql://postgres:pass@127.0.0.1:5432/postgres_dev
META_DB_URL=postgresql://postgres:pass@127.0.0.1:5432/meta_db_dev
SHARED_DB_URL=postgresql://postgres:pass@127.0.0.1:5432/shared_db_dev
BASE_TENANT_DB_URL=postgresql://postgres:pass@127.0.0.1:5432
```

### Commands to re-copy production → dev anytime

```bash
sudo -u postgres pg_dump -d postgres  | sudo -u postgres psql -d postgres_dev
sudo -u postgres pg_dump -d meta_db   | sudo -u postgres psql -d meta_db_dev
sudo -u postgres pg_dump -d shared_db | sudo -u postgres psql -d shared_db_dev
```
