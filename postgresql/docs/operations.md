# Operations

Debee provides composable operations that can be run individually or combined. Each operation delegates to external tools and your SQL scripts — debee itself contains no database logic.

## Operations Overview

| Operation | What It Does |
|-----------|-------------|
| `recreateDatabase` | Runs `psql -f $DBRECREATESCRIPT` — executes your recreation SQL |
| `restoreDatabase` | Runs `pg_restore $DBBACKUPFILE` — restores your backup |
| `preUpdateScripts` | Runs `psql -f $DBPREUPDATESCRIPTS` — executes your pre-migration scripts |
| `updateDatabase` | Runs `psql -f 001_*.sql`, `002_*.sql`, etc. — applies your migrations in order |
| `postUpdateScripts` | Runs `psql -f $DBPOSTUPDATESCRIPTS` — executes your post-migration scripts |
| `prepareVersionTable` | Generates database object documentation (see [Version Table](version-table.md)) |
| `execSql` | Executes a SQL file, inline SQL, or opens interactive psql |
| `runTests` | Runs test files and suites (see [Testing](testing.md)) |
| `fullService` | Runs all operations in sequence: recreate → restore → pre-update → update → post-update |

## Running Operations

```powershell
# Single operation
.\debee.ps1 -Operations updateDatabase

# Multiple operations
.\debee.ps1 -Operations @("restoreDatabase", "updateDatabase")

# Full pipeline
.\debee.ps1 -Operations fullService
```

```bash
# Bash / Python equivalent
./debee.sh -o updateDatabase
./debee.sh -o restoreDatabase,updateDatabase
python debee.py -o fullService
```

## The fullService Flow

When you run `fullService`, debee executes this sequence:

1. **Loads environment** → reads configuration files
2. **Recreates database** → executes `psql -f $DBRECREATESCRIPT` (your DROP/CREATE commands)
3. **Restores backup** → executes `pg_restore $DBBACKUPFILE` (your data and schema)
4. **Runs pre-updates** → executes `psql -f $DBPREUPDATESCRIPTS` (your preparation logic)
5. **Applies migrations** → finds and executes `XXX_*.sql` files in order (your ALTER/CREATE statements)
6. **Runs post-updates** → executes `psql -f $DBPOSTUPDATESCRIPTS` (your cleanup logic)

## Migration File Naming Convention

Migration files must follow the pattern: `XXX_description.sql`

- `XXX` = exactly 3 digits (e.g., `001`, `060`, `999`)
- `_` = underscore separator
- `description` = any descriptive name
- `.sql` = required extension

**Valid examples:**
- `001_create_users_table.sql`
- `060_add_indexes.sql`
- `999_examples.sql`

**Invalid examples:**
- `ABC_test.sql` — prefix is not digits
- `01_test.sql` — prefix is not exactly 3 digits
- `ALL_TASKS_COMPLETE.md` — wrong extension

Files that don't match the pattern are skipped with an informative message.

## Migration Range Selection

Control which migrations run using start and end numbers:

```powershell
# Run only migrations 010 through 020
.\debee.ps1 -Operations updateDatabase -UpdateStartNumber 10 -UpdateEndNumber 20
```

```bash
# Bash equivalent
./debee.sh -o updateDatabase -s 10 -n 20

# Python equivalent
python debee.py -o updateDatabase -s 10 -n 20
```

When no range is specified, all migration files in the directory are processed.

Range can also be set via environment variables `DBUPDATESTARTNUMBER` and `DBUPDATEENDNUMBER` in your `.env` file. CLI arguments override environment variables.

## Ad-Hoc Scripts

The `DBADHOCDIRECTORY` environment variable points to a directory of emergency/hotfix scripts. These scripts:

- Are scanned recursively for SQL files
- Have no naming convention requirement — any `.sql` file is processed
- Are tracked separately in [version table](version-table.md) output with an "Ad-hoc" source label
- Enable emergency fixes while maintaining change history

```bash
# Set in debee.env
DBADHOCDIRECTORY=ad-hoc-scripts/
```

**Use cases:** emergency fixes, hotfix deployment, production repairs, data corrections.

## execSql Operation

Execute arbitrary SQL without going through the migration pipeline:

```powershell
# Execute a SQL file
.\debee.ps1 -Operations execSql -SqlFile path/to/script.sql

# Execute inline SQL
.\debee.ps1 -Operations execSql -Sql "SELECT version();"

# Open interactive psql session
.\debee.ps1 -Operations execSql
```

```bash
# Bash equivalents
./debee.sh -o execSql --sql-file path/to/script.sql
./debee.sh -o execSql --sql "SELECT version();"
./debee.sh -o execSql
```

Priority: file → inline command → interactive session.

## Pre/Post Update Scripts

Configure scripts that run before or after migrations. Multiple scripts can be specified with semicolon separators:

```bash
# debee.env
DBPREUPDATESCRIPTS=hooks/pre1.sql;hooks/pre2.sql
DBPOSTUPDATESCRIPTS=hooks/post.sql
```

## Backup Formats

The `restoreDatabase` operation supports multiple backup formats via `DBBACKUPTYPE`:

| Format | Description |
|--------|-------------|
| `custom` | pg_dump custom archive format (`.dump`) |
| `dir` | pg_dump directory format |
| `file` | Plain SQL file |
