# Debee — PostgreSQL Migration Orchestrator

Debee orchestrates PostgreSQL database operations — migrations, backups, testing, and documentation — without embedding any SQL itself. Your database logic stays in SQL files where it belongs. Debee just runs them in the right order.

## Philosophy: Orchestration, Not Implementation

Debee **does not** create databases, define schemas, or contain SQL statements.

Debee **does** read configuration, execute your SQL scripts in order, call PostgreSQL tools (psql, pg_restore), and manage the execution flow.

```
┌─────────────────┐
│     debee.*      │  ← Orchestrator (ps1 / sh / py)
└────────┬────────┘
         │
         ├── Reads: debee.env
         ├── Calls: psql -f recreate_script.sql
         ├── Calls: pg_restore backup.dump
         ├── Calls: psql -f 001_init.sql
         ├── Calls: psql -f 002_tables.sql
         ├── Calls: psql -f 003_functions.sql
         └── Calls: psql -f post_update.sql
```

**Debee is a conductor, not a musician.** It doesn't know your schema — it just runs the scripts you provide in the right order with the right tools.

## Features

- **Operations pipeline** — recreate, restore, migrate, pre/post hooks, ad-hoc scripts, all composable
- **Numbered migrations** — `XXX_description.sql` files applied in order with range selection
- **Test runner** — flat test files, suite directories, transaction/database isolation modes
- **Version table** — track every database object across migrations with JSON, Markdown, CSV, or interactive HTML output
- **Cross-platform** — PowerShell, Bash, and Python implementations with identical interfaces
- **Configuration-driven** — everything controlled through `.env` files with environment-specific overrides

## Quick Start

**1. Create a configuration file:**

```bash
# debee.env
PGHOST=localhost
PGPORT=5432
PGUSER=postgres
PGPASSWORD=secret
DBDESTDB=myapp_db
DBRECREATESCRIPT=scripts/recreate.sql
DBUPDATESTARTNUMBER=1
DBUPDATEENDNUMBER=100
```

**2. Run migrations:**

```bash
# PowerShell
.\debee.ps1 -Operations updateDatabase

# Bash
./debee.sh -o updateDatabase

# Python
python debee.py -o updateDatabase
```

**3. Run everything (recreate → restore → migrate):**

```bash
.\debee.ps1 -Operations fullService
```

## Platform Compatibility

| Orchestrator | Platform | Runtime Required |
|-------------|----------|------------------|
| `debee.ps1` | Windows, Linux*, macOS* | PowerShell 5.1+ or Core 6.0+ |
| `debee.sh`  | Linux, macOS, WSL | Bash 4.0+ |
| `debee.py`  | Windows, Linux, macOS | Python 3.6+ |

\* With PowerShell Core installed

## Documentation

| Topic | Description |
|-------|-------------|
| [Configuration](docs/configuration.md) | Environment files, variables reference, local overrides |
| [Operations](docs/operations.md) | All operations explained, migration naming, ad-hoc scripts |
| [Testing](docs/testing.md) | Test runner, suites, isolation modes, shared setup, ordering |
| [Version Table](docs/version-table.md) | Database object tracking, output formats, HTML dashboard |
| [Cross-Platform](docs/cross-platform.md) | CLI reference for ps1/sh/py, platform guidance |
| [Changelog](CHANGELOG.md) | Version history and migration notes |
