# Configuration

Debee is entirely configuration-driven. All behavior is controlled through environment variables loaded from `.env` files.

## Environment File Loading

Debee loads configuration from multiple files in order, with later files overriding earlier values:

1. **`debee.env`** — Main configuration (committed to version control)
2. **`.debee.env`** — Local overrides (gitignored, for developer-specific settings)
3. **`debee.<environment>.env`** — Environment-specific overrides (when `-e`/`--environment` is specified)

### Example

```bash
# Load base config only
./debee.sh -o updateDatabase

# Load base config + staging overrides (debee.staging.env)
./debee.sh -e staging -o updateDatabase
```

## Environment Variables Reference

### Connection Settings

| Variable | Purpose | Example |
|----------|---------|---------|
| `PGHOST` | PostgreSQL host | `localhost` |
| `PGPORT` | PostgreSQL port | `5432` |
| `PGUSER` | PostgreSQL user | `postgres` |
| `PGPASSWORD` | PostgreSQL password | `secret` |
| `DBCONNECTDB` | Admin database for connections (used for recreate/restore) | `postgres` |
| `DBDESTDB` | Target database name | `myapp_db` |

### Tool Paths

| Variable | Purpose | Example |
|----------|---------|---------|
| `DBPSQLFILE` | Path to psql executable | `psql` |
| `DBPGRESTOREFILE` | Path to pg_restore executable | `pg_restore` |

### Script Paths

| Variable | Purpose | Example |
|----------|---------|---------|
| `DBRECREATESCRIPT` | Path to your database recreation SQL | `scripts/recreate.sql` |
| `DBBACKUPFILE` | Path to your database backup | `backups/prod.dump` |
| `DBBACKUPTYPE` | Backup format | `custom`, `dir`, `file` |
| `DBPREUPDATESCRIPTS` | Pre-update scripts (semicolon-separated) | `hooks/pre1.sql;hooks/pre2.sql` |
| `DBPOSTUPDATESCRIPTS` | Post-update scripts (semicolon-separated) | `hooks/post.sql` |
| `DBADHOCDIRECTORY` | Directory containing ad-hoc/hotfix scripts | `ad-hoc-scripts/` |

### Migration Range

| Variable | Purpose | Example |
|----------|---------|---------|
| `DBUPDATESTARTNUMBER` | First migration number to run | `1` |
| `DBUPDATEENDNUMBER` | Last migration number to run | `100` |

### Version Table

| Variable | Purpose | Example |
|----------|---------|---------|
| `DBVERSIONTABLEFORMATS` | Output formats (semicolon-separated) | `html;json;md;csv` |

## Local Overrides Pattern

Use `.debee.env` for settings that vary per developer (credentials, local paths):

```bash
# .debee.env (gitignored)
PGPASSWORD=my_local_password
PGHOST=localhost
DBBACKUPFILE=/home/me/backups/latest.dump
```

This keeps secrets out of version control while sharing the base configuration with the team.

## Example debee.env

```bash
# debee.env — base configuration

# Connection
PGHOST=localhost
PGPORT=5432
PGUSER=postgres
DBCONNECTDB=postgres
DBDESTDB=myapp_db

# Scripts
DBRECREATESCRIPT=scripts/recreate_database.sql
DBBACKUPFILE=backups/production.dump
DBBACKUPTYPE=custom
DBPREUPDATESCRIPTS=hooks/pre_update.sql
DBPOSTUPDATESCRIPTS=hooks/post_update.sql
DBADHOCDIRECTORY=ad-hoc-scripts/

# Migration range
DBUPDATESTARTNUMBER=1
DBUPDATEENDNUMBER=999

# Version table output
DBVERSIONTABLEFORMATS=html;json;md

# Tool paths (defaults work if tools are on PATH)
DBPSQLFILE=psql
DBPGRESTOREFILE=pg_restore
```

## Example Project Structure

```
project/
├── debee.ps1                    # The orchestrator
├── debee.env                    # Shared configuration
├── .debee.env                   # Local overrides (gitignored)
├── debee.staging.env            # Staging environment config
│
├── scripts/
│   └── recreate_database.sql    # Your recreation logic
│
├── backups/
│   └── production.dump          # Your database backup
│
├── migrations/
│   ├── 001_initial_schema.sql   # Your migrations
│   ├── 002_add_users.sql
│   └── 003_add_indexes.sql
│
├── hooks/
│   ├── pre_update.sql           # Your pre-update logic
│   └── post_update.sql          # Your post-update logic
│
└── ad-hoc-scripts/
    └── hotfix_permissions.sql   # Emergency fixes
```
