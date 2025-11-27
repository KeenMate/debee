# Debee - Database Migration Orchestrator

## Project Overview
Debee is a cross-platform database migration orchestrator for PostgreSQL. It provides PowerShell, Bash, and Python implementations for running database operations including recreation, restoration, and migration scripts.

## Directory Structure
```
debee/
├── postgresql/           # PostgreSQL-specific tools
│   ├── debee.ps1        # PowerShell orchestrator (Windows primary)
│   ├── debee.sh         # Bash orchestrator (Linux/macOS)
│   ├── debee.py         # Python orchestrator (cross-platform)
│   ├── extract-db-objects.py  # Database object extractor
│   ├── version_table_template.html  # HTML dashboard template
│   ├── CHANGELOG.md     # Version history
│   └── README.md        # Documentation
```

## Key Concepts

### Migration File Naming Convention
Files must follow the pattern: `XXX_description.sql`
- `XXX` = exactly 3 digits (e.g., `001`, `060`, `999`)
- `_` = underscore separator
- `description` = any descriptive name
- `.sql` = required extension

Examples:
- `001_create_users_table.sql` ✓
- `060_add_indexes.sql` ✓
- `ALL_TASKS_COMPLETE.md` ✗ (not 3 digits, wrong extension)

### Operations
- `recreateDatabase` - Drop and recreate database
- `restoreDatabase` - Restore from backup
- `updateDatabase` - Apply migration scripts
- `preUpdateScripts` - Run pre-migration scripts
- `postUpdateScripts` - Run post-migration scripts
- `prepareVersionTable` - Generate database object documentation
- `fullService` - Run all operations in sequence

### Environment Configuration
Uses `.env` files for configuration:
- `debee.env` - Main configuration
- `.debee.env` - Local overrides (gitignored)
- `debee.<environment>.env` - Environment-specific

Key variables:
- `PGHOST`, `PGPORT`, `PGUSER`, `PGPASSWORD` - PostgreSQL connection
- `DBDESTDB` - Target database name
- `DBUPDATESTARTNUMBER`, `DBUPDATEENDNUMBER` - Migration range

## Common Tasks

### Running Migrations
```powershell
# Run specific migration range
.\debee.ps1 -Operations updateDatabase -UpdateStartNumber 60 -UpdateEndNumber 62

# Full service (recreate, restore, migrate)
.\debee.ps1 -Operations fullService
```

### File Filtering Logic
The `Get-FilesByNumericPrefix` function in `debee.ps1`:
1. Uses `Get-ChildItem -Filter "???_*"` for initial file discovery
2. Validates files match `^\d{3}_.*\.sql$` pattern
3. Extracts numeric prefix and checks against range
4. Skips files that don't match pattern with informative message

## Testing Changes
When modifying orchestrators, test with:
1. Valid migration files (e.g., `060_test.sql`)
2. Invalid files (e.g., `ABC_test.sql`, `test.md`)
3. Edge cases (e.g., `000_`, `999_`)
