## SQL Firewall for PostgreSQL

`sql_firewall` is an extension for PostgreSQL that provides multi-layered protection against SQL injection, unauthorized data access, and misuse of database services.

This extension intercepts incoming SQL queries and analyzes them based on predefined rules, policies, and limits. It enhances database security by allowing only approved or safe query patterns to be executed.

---

### 🚀 Features

#### Multiple Operating Modes:
- **learn**: Logs all new queries to a rule table without blocking them, helping to build an initial ruleset.
- **permissive**: Logs and warns about unknown queries based on the hash ruleset but does not block them. Blocks queries that match regex rules.
- **enforce**: Allows only previously approved queries to be executed and blocks all others, providing the highest level of security.

#### Rule-Based Filtering:
Recognizes queries using fingerprinting (hashing) and enforces per-role and per-database rules.

#### Regex-Based Threat Detection:
Blocks queries that match custom regular expression patterns. This is highly effective against common SQL injection (SQLi) techniques and other malicious patterns. The matching is case-insensitive.

#### Keyword Blacklist:
Automatically blocks queries containing dangerous SQL keywords such as `DROP`, `TRUNCATE`, etc.

#### Rate Limiting:
Limits the total number of queries or specific query types (e.g., `SELECT`, `UPDATE`) per user in a specified time frame.

#### Quiet Hours:
Blocks query execution during specified periods, such as night hours, when database activity is not expected.

#### Application Blocking:
Blocks connections based on the `application_name` parameter.

#### Detailed Logging:
Logs every allowed, blocked, or learned query with detailed activity information.

---

### 📦 Requirements

- **PostgreSQL Version**: 9.6 or higher
- **Build Tools**: Standard C build tools (e.g., `make`, `gcc`)
- **PostgreSQL Development Files**: Required for compilation (e.g., `postgresql-server-dev-16` or `postgresql-devel`)

---

### ⚙️ Installation

**Clone the Repository:**
```bash
git clone https://github.com/your_user/sql_firewall.git
cd sql_firewall
```

**Build and Install:**
Ensure `pg_config` is available in your system `PATH`.
```bash
make
sudo make install
```

**Activate the Extension:**
Connect to the target database and run:
```sql
CREATE EXTENSION sql_firewall;
```
This will create the required tables: `sql_firewall_rules`, `sql_firewall_activity_log`, and `sql_firewall_regex_rules`.

---

### 🛠️ Configuration

Several GUC parameters can be set in `postgresql.conf` or using `ALTER SYSTEM`:

| Parameter | Description | Default |
|----------|-------------|---------|
| sql_firewall.mode | Firewall mode (`learn`, `permissive`, `enforce`) | learn |
| sql_firewall.enable_regex_scan | Enable regex-based threat detection | true |
| sql_firewall.enable_keyword_scan | Enable blacklist keyword scanning | false |
| sql_firewall.blacklisted_keywords | Comma-separated keywords to block | drop,truncate |
| sql_firewall.enable_quiet_hours | Enable quiet hours feature | false |
| sql_firewall.quiet_hours_start | Quiet hours start (HH:MM) | 22:00 |
| sql_firewall.quiet_hours_end | Quiet hours end (HH:MM) | 06:00 |
| sql_firewall.enable_rate_limiting | Enable general query rate limiting | false |
| sql_firewall.rate_limit_count | Max queries allowed in rate_limit_seconds | 100 |
| sql_firewall.rate_limit_seconds | Time window for rate limiting (seconds) | 60 |
| sql_firewall.command_limit_seconds | Time window for command-based limits (seconds) | 60 |
| sql_firewall.select_limit_count | SELECT query limit (0 = unlimited) | 0 |
| sql_firewall.insert_limit_count | INSERT query limit (0 = unlimited) | 0 |
| sql_firewall.update_limit_count | UPDATE query limit (0 = unlimited) | 0 |
| sql_firewall.delete_limit_count | DELETE query limit (0 = unlimited) | 0 |
| sql_firewall.enable_application_blocking | Enable blocking by application name | false |
| sql_firewall.blocked_applications | Comma-separated list of blocked `application_name`s | '' (empty) |

**Example:**
```sql
-- Set mode to 'enforce' and enable regex scanning
ALTER SYSTEM SET sql_firewall.mode = 'enforce';
ALTER SYSTEM SET sql_firewall.enable_regex_scan = 'on';

-- Reload configuration
SELECT pg_reload_conf();
```

---

### 🚦 Usage

#### 1. Learn Mode
On installation, firewall defaults to `learn` mode. All unseen queries are saved in `sql_firewall_rules` with `is_approved = false`.

#### 2. Approving Rules
Review and approve safe query patterns:
```sql
UPDATE sql_firewall_rules SET is_approved = true WHERE rule_id = 123;
-- or approve by role
UPDATE sql_firewall_rules SET is_approved = true WHERE role_name = 'app_user';
```

#### 3. Using Regex Rules
Enable feature and add patterns:
```sql
ALTER SYSTEM SET sql_firewall.enable_regex_scan = 'on';

INSERT INTO sql_firewall_regex_rules (pattern, description, action)
VALUES ('or\s+1\s*=\s*1', 'Classic SQLi attack vector', 'BLOCK');
```

#### 4. Enforce Mode
```sql
ALTER SYSTEM SET sql_firewall.mode = 'enforce';
SELECT pg_reload_conf();
```

---

### 🗃️ Database Tables

#### `sql_firewall_rules`
Stores learned/approved query rules.
- rule_id
- role_name
- database_name
- command_type
- query_fingerprint
- is_approved
- created_at

#### `sql_firewall_regex_rules`
Stores regex patterns for malicious query detection.
- id
- pattern
- description
- action (BLOCK or ALLOW)
- is_active
- created_at

#### `sql_firewall_activity_log`
Audit log for firewall activity.
- log_id
- log_time
- role_name
- database_name
- action (ALLOWED, BLOCKED, LEARNED)
- reason
- query_text
- command_type

