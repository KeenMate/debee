# Debee - PostgreSQL Migration Orchestrator

## What is Debee?

`debee.ps1` is a PowerShell orchestration script that coordinates PostgreSQL database operations. **It doesn't contain any database logic itself** - instead, it acts as a conductor that executes external SQL scripts and PostgreSQL tools in a specific order.

## Key Concept: Orchestration, Not Implementation

Debee **does not**:
- Create databases directly
- Define table schemas
- Contain SQL statements
- Know your database structure

Debee **does**:
- Read configuration from environment files
- Execute external SQL scripts in order
- Call PostgreSQL tools (psql, pg_restore)
- Manage execution flow and error handling

## How It Works

```
┌─────────────────┐
│   debee.ps1     │  <── Orchestrator
└────────┬────────┘
         │
         ├── Reads: debee.env
         │
         ├── Calls: psql -f recreate_script.sql
         │
         ├── Calls: pg_restore backup.dump
         │
         ├── Calls: psql -f 001_init.sql
         ├── Calls: psql -f 002_tables.sql
         ├── Calls: psql -f 003_functions.sql
         │
         └── Calls: psql -f post_update.sql
```

## Configuration-Driven Execution

Everything debee does is controlled by environment variables:

```bash
# debee.env

# Connection settings (passed to psql/pg_restore)
PGHOST=localhost
PGUSER=postgres
PGDATABASE=myapp

# Scripts that debee will execute
DBRECREATESCRIPT=database/recreate.sql      # YOUR script for recreation
DBBACKUPFILE=/backups/prod.dump            # YOUR backup file
DBPREUPDATESCRIPTS=hooks/pre.sql           # YOUR pre-update script
DBPOSTUPDATESCRIPTS=hooks/post.sql         # YOUR post-update script
DBADHOCDIRECTORY=ad-hoc-scripts/           # YOUR ad-hoc scripts directory

# Tools debee will call
DBPSQLFILE=psql                            # PostgreSQL client
DBPGRESTOREFILE=pg_restore                 # PostgreSQL restore tool
```

## The Orchestration Flow

When you run `.\debee.ps1 -Operations fullService`, it:

1. **Loads environment** → Reads configuration files
2. **Recreates database** → Executes `psql -f $DBRECREATESCRIPT`
   - The actual DROP/CREATE DATABASE commands are in YOUR script
3. **Restores backup** → Executes `pg_restore $DBBACKUPFILE`
   - The data and schema come from YOUR backup
4. **Runs pre-updates** → Executes `psql -f $DBPREUPDATESCRIPTS`
   - Any preparation logic is in YOUR scripts
5. **Applies migrations** → Finds and executes `XXX_*.sql` files in order
   - The actual ALTER/CREATE statements are in YOUR migration files
6. **Runs post-updates** → Executes `psql -f $DBPOSTUPDATESCRIPTS`
   - Any cleanup logic is in YOUR scripts

## Example: What Debee Doesn't Do

```powershell
# debee.ps1 does NOT contain code like this:
# CREATE DATABASE myapp;
# CREATE TABLE users (...);

# Instead, it does this:
& $Env:DBPSQLFILE -f $Env:DBRECREATESCRIPT  # Runs YOUR SQL file
```

## Example: Complete Setup

```
project/
├── debee.ps1                    # The orchestrator
├── debee.env                    # Configuration
│
├── scripts/
│   └── recreate_database.sql   # YOUR recreation logic
│
├── backups/
│   └── production.dump         # YOUR database backup
│
├── migrations/
│   ├── 001_initial_schema.sql  # YOUR migrations
│   ├── 002_add_users.sql
│   └── 003_add_indexes.sql
│
└── hooks/
    ├── pre_update.sql          # YOUR pre-update logic
    └── post_update.sql         # YOUR post-update logic
```

## Why This Design?

1. **Separation of Concerns**: Orchestration logic is separate from database logic
2. **Flexibility**: You control all SQL - debee just runs it
3. **Reusability**: Same orchestrator works for any PostgreSQL project
4. **Transparency**: All database changes are in SQL files you can review
5. **Version Control**: SQL files can be tracked separately from the tool

## Operations

Each operation just calls different external resources:

| Operation | What It Actually Does |
|-----------|----------------------|
| `recreateDatabase` | Runs: `psql -f $DBRECREATESCRIPT` |
| `restoreDatabase` | Runs: `pg_restore $DBBACKUPFILE` |
| `updateDatabase` | Runs: `psql -f 001_*.sql`, `psql -f 002_*.sql`, etc. |
| `preUpdateScripts` | Runs: `psql -f $DBPREUPDATESCRIPTS` |
| `postUpdateScripts` | Runs: `psql -f $DBPOSTUPDATESCRIPTS` |
| `fullService` | Runs all of the above in sequence |

## Usage Examples

```powershell
# Let debee orchestrate everything
.\debee.ps1 -Operations fullService

# Just run your migration files
.\debee.ps1 -Operations updateDatabase

# Run specific migration range (files 010_*.sql through 020_*.sql)
.\debee.ps1 -Operations updateDatabase -UpdateStartNumber 10 -UpdateEndNumber 20

# Multiple operations
.\debee.ps1 -Operations @("restoreDatabase", "updateDatabase")
```

## Environment Variables Reference

| Variable | Purpose | Example |
|----------|---------|---------|
| `PGHOST` | PostgreSQL host | `localhost` |
| `PGPORT` | PostgreSQL port | `5432` |
| `PGUSER` | PostgreSQL user | `postgres` |
| `PGPASSWORD` | PostgreSQL password | `secret` |
| `DBCONNECTDB` | Admin database for connections | `postgres` |
| `DBDESTDB` | Target database name | `myapp_db` |
| `DBRECREATESCRIPT` | Path to YOUR recreation SQL | `scripts/recreate.sql` |
| `DBBACKUPFILE` | Path to YOUR backup | `backups/prod.dump` |
| `DBBACKUPTYPE` | Backup format | `custom`, `dir`, `file` |
| `DBPREUPDATESCRIPTS` | Pre-update scripts (semicolon-separated) | `hooks/pre1.sql;hooks/pre2.sql` |
| `DBPOSTUPDATESCRIPTS` | Post-update scripts | `hooks/post.sql` |
| `DBADHOCDIRECTORY` | Directory containing ad-hoc scripts | `ad-hoc-scripts/` |
| `DBUPDATESTARTNUMBER` | First migration to run | `1` |
| `DBUPDATEENDNUMBER` | Last migration to run | `100` |

## The Bottom Line

**Debee is a conductor, not a musician.** It doesn't know how to create your database or what your schema looks like - it just knows how to run the scripts you provide in the right order with the right tools.

Your database logic stays in SQL files where it belongs. Debee just orchestrates their execution.