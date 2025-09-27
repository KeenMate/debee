# Debee - Database Migration and Management Tools

A collection of database migration orchestration and management tools, currently supporting PostgreSQL with potential for expansion to other databases.

## Project Structure

```
debee/
├── README.md                 # This file
├── debee/
│   └── version_table_template.html # Interactive HTML template
└── postgresql/               # PostgreSQL-specific tools
    ├── debee.ps1            # Migration orchestrator script
    ├── extract-db-objects.py # Database object extractor
    └── DEBEE_ORCHESTRATOR.md # Detailed orchestrator documentation
```

## Database Support

### PostgreSQL
Complete toolset for PostgreSQL database migration management. See [`postgresql/`](postgresql/) directory for:
- Migration orchestration
- Database object tracking
- Environment-based configuration

## Tools Overview

### 1. PostgreSQL Migration Orchestrator (`postgresql/debee.ps1`)

A PowerShell orchestration script that coordinates PostgreSQL database operations. **It doesn't contain any database logic itself** - instead, it executes external SQL scripts and PostgreSQL tools in a specific order.

#### Key Features

- **Pure Orchestration**: No embedded SQL - all database logic lives in external scripts
- **Environment-based Configuration**: Supports multiple environments through `.env` files
- **Multiple Operations**: Restore, recreate, update database with pre/post scripts
- **Numbered Migration Files**: Processes SQL files with numeric prefixes (e.g., `001_init.sql`, `002_users.sql`)

#### Quick Usage

```powershell
cd postgresql
.\debee.ps1 -Environment <env> -Operations <operation> [-UpdateStartNumber <num>] [-UpdateEndNumber <num>]
```

#### Parameters

- **Environment** (optional): Environment name for configuration file selection
  - Uses `debee.<env>.env` and `.debee.<env>.env` files
  - If not specified, uses `debee.env` and `.debee.env`

- **Operations** (required): Database operations to perform
  - `restoreDatabase`: Restore database from backup
  - `recreateDatabase`: Drop and recreate database
  - `updateDatabase`: Apply migration scripts
  - `preUpdateScripts`: Run pre-update scripts
  - `postUpdateScripts`: Run post-update scripts
  - `prepareVersionTable`: Generate comprehensive database object documentation
  - `fullService`: Run all operations in sequence (default)

- **UpdateStartNumber** (optional): Starting migration file number (default: -1 for all)
- **UpdateEndNumber** (optional): Ending migration file number (default: -1 for all)

#### Examples

```powershell
cd postgresql

# Run full service for production environment
.\debee.ps1 -Environment prod -Operations fullService

# Only restore database for dev environment
.\debee.ps1 -Environment dev -Operations restoreDatabase

# Update database with specific migration range
.\debee.ps1 -Operations updateDatabase -UpdateStartNumber 10 -UpdateEndNumber 20

# Run multiple operations
.\debee.ps1 -Operations @("restoreDatabase", "updateDatabase")

# Generate database object documentation
.\debee.ps1 -Operations prepareVersionTable
```

For detailed information about how the orchestrator works, see [`postgresql/DEBEE_ORCHESTRATOR.md`](postgresql/DEBEE_ORCHESTRATOR.md).

#### Environment File Configuration

Create a `debee.env` file (or `debee.<environment>.env`) with the following variables:

```bash
# PostgreSQL connection settings
PGHOST=localhost
PGPORT=5432
PGUSER=postgres
PGPASSWORD=password

# Database names
DBCONNECTDB=postgres          # Database to connect for admin operations
DBDESTDB=myapp_db            # Target database name

# Paths to PostgreSQL tools
DBPSQLFILE=psql
DBPGRESTOREFILE=pg_restore

# Backup settings
DBBACKUPFILE=/path/to/backup.dump
DBBACKUPTYPE=custom          # Options: file, dir, custom
DBRESTOREJOBCOUNT=4          # Parallel jobs for restore
DBCREATEONRESTORE=false      # Create database during restore

# Script paths
DBRECREATESCRIPT=recreate_db.sql
DBPREUPDATESCRIPTS=pre_update1.sql;pre_update2.sql
DBPOSTUPDATESCRIPTS=post_update.sql
DBADHOCDIRECTORY=ad-hoc-scripts/

# Migration range (optional)
DBUPDATESTARTNUMBER=1
DBUPDATEENDNUMBER=100

# Version table configuration
DBVERSIONTABLEFORMATS=json;md;html    # Semicolon-separated export formats
DBVERSIONTABLEOUTPUTFOLDER=.          # Output directory for version table files
DBVERSIONTABLEFILENAME=db-objects     # Base filename (extensions added automatically)
```

#### Migration File Naming Convention

Migration files should follow the pattern:
- `XXX_description.sql` where XXX is a 3-digit number (e.g., `001_init.sql`)
- Files are processed in numeric order
- Special patterns: `9X_` files and `999-examples.sql` are also supported

#### Database Object Documentation Generation

The orchestrator includes integrated database object documentation generation through the `prepareVersionTable` operation. This feature automatically analyzes your migration files and produces comprehensive documentation of your database schema evolution.

**What it generates:**
- `db-objects.json`: Machine-readable database object inventory with complete change history
- `db-objects.md`: Human-readable markdown table showing current database schema state

**Key benefits:**
- **Automated Documentation**: No manual maintenance of schema documentation required
- **Change Tracking**: Complete history of when and where each database object was modified
- **Migration and Ad-hoc Separation**: Clear distinction between planned migrations and emergency fixes
- **Schema Evolution Visibility**: Easily see how your database structure has evolved over time
- **Team Collaboration**: Standardized documentation format for team visibility

**Usage example:**
```bash
# Generate complete database object documentation
./debee.sh -o prepareVersionTable

# Files created:
# - db-objects.json (detailed object history)
# - db-objects.md (formatted table for documentation)
```

This operation leverages the database object extractor to provide a consolidated view of your database structure, making it invaluable for database maintenance, auditing, and team onboarding.

#### Interactive HTML Dashboard (NEW in v3.0.0)

Debee now provides a powerful interactive HTML dashboard for database object analysis, offering a professional, responsive interface that goes far beyond static documentation.

**Setup:**
1. Ensure the template is available in your project:
   ```bash
   mkdir -p debee
   cp debee/version_table_template.html debee/
   ```

2. Configure HTML output in your environment file:
   ```bash
   DBVERSIONTABLEFORMATS=json;md;html
   ```

3. Generate the interactive dashboard:
   ```bash
   ./debee.sh -o prepareVersionTable
   ```

**🎯 Advanced Features:**

**Professional UI Design:**
- **Audi-inspired aesthetics**: Clean, sophisticated design suitable for enterprise environments
- **Responsive architecture**: Seamless experience from desktop to mobile devices
- **Icon-based controls**: UTF-8 symbols for universal compatibility and touch-friendly interaction

**Real-time Filtering System (8 Independent Filters):**
- **Schema Filter**: Text search for specific schemas
- **Object Name Filter**: Find objects by name pattern
- **Object Type Filter**: Dropdown with all detected types (table, function, view, etc.)
- **Operation Filter**: Filter by database operations (CREATE, ALTER, DROP, CREATE_OR_REPLACE)
- **Last File Filter**: Search for specific migration files
- **Migration Updates Filter**: Find objects modified by specific migration scripts
- **Ad-hoc Updates Filter**: Track emergency/hotfix modifications
- **Update Count Filter**: Find frequently modified objects (minimum threshold)

**Enhanced Data Visualization:**
- **File:Line:Operation format**: Precise location and operation type for every change
- **Color-coded object types**: Visual distinction for quick identification
- **Clickable elements**: Click any data point to instantly filter results
- **Complete change history**: Full migration and ad-hoc update tracking per object

**Mobile-First Responsive Design:**
- **Card-based mobile layout**: Touch-optimized interface for phones and tablets
- **Adaptive breakpoints**: 4 responsive layouts for different screen sizes
- **Natural scrolling**: Mobile-friendly infinite scroll instead of constrained containers
- **Filter toggle**: Show/hide filters on mobile to maximize content viewing space

**Smart Interface Features:**
- **Local storage preferences**: Layout settings persist across sessions and files
- **Dynamic height management**: Optimized viewport utilization (55vh with filters, 72vh without)
- **Conditional ad-hoc support**: Interface automatically adapts if no ad-hoc scripts are present
- **Real-time statistics**: Live updates showing filtered vs total object counts

**Professional Controls:**
- **Layout toggle**: Switch between contained (◧) and full-width (◨) modes
- **Filter visibility**: Show (▼) or hide (▲) filter panel for maximum data viewing
- **Consistent positioning**: Right-aligned controls across all device sizes

The HTML dashboard transforms database documentation from static tables into an interactive analysis platform, making it invaluable for schema evolution tracking, database auditing, team collaboration, and project onboarding.

### 2. PostgreSQL Database Object Extractor (`postgresql/extract-db-objects.py`)

A Python script that analyzes SQL migration files to extract and track PostgreSQL database objects (tables, functions, indexes, etc.) across all migrations.

#### Features

- **Object Detection**: Identifies tables, functions, procedures, indexes, views, triggers, and schemas
- **Change Tracking**: Tracks all modifications to each object across migration files
- **Multiple Output Formats**: JSON, CSV, or Markdown output
- **Schema Support**: Handles schema-qualified object names
- **Operation Tracking**: Tracks CREATE, ALTER, DROP operations
- **Ad-hoc Script Support**: Scans additional directory for emergency/hotfix scripts (marked with AD-HOC prefix)

#### Usage

```bash
cd postgresql
python extract-db-objects.py [--format {json,csv,markdown}] [--output <file>]
```

#### Parameters

- **--format**: Output format (default: json)
  - `json`: JSON format with full details
  - `csv`: CSV format for spreadsheet import
  - `markdown`: Markdown table for documentation
  - `html`: Interactive HTML with filtering (requires template)

- **--output**: Output file path (default: stdout)

#### Examples

```bash
# Extract objects to JSON file
python extract-db-objects.py --format json --output db_objects.json

# Generate CSV report
python extract-db-objects.py --format csv --output db_objects.csv

# Create Markdown documentation
python extract-db-objects.py --format markdown --output DB_OBJECTS.md

# Generate interactive HTML (requires debee/version_table_template.html)
python extract-db-objects.py --format html --output db_objects.html

# Display JSON to console
python extract-db-objects.py

# Include ad-hoc scripts from specific directory
DBADHOCDIRECTORY=hotfix-scripts/ python extract-db-objects.py --format markdown --output db_objects.md
```

#### Output Information

The script provides:
- **Object Identification**: Schema, name, and type of each database object
- **Latest Version**: File and line number of the most recent update
- **Update History**: Complete history of all modifications
- **Summary Statistics**: Objects grouped by type and schema
- **Ad-hoc Tracking**: Scripts from emergency/hotfix directory marked with (AD-HOC) prefix
- **Update Classification**: Separate counts for migration vs ad-hoc updates

#### Sample Output (JSON)

```json
[
  {
    "schema": "public",
    "object_name": "users",
    "object_type": "table",
    "last_update_file": "003_user_updates.sql",
    "last_update_line": 45,
    "last_update_source": "migration",
    "total_updates": 3,
    "all_updates": [
      {"file": "001_init.sql", "line": 10, "operation": "CREATE", "source": "migration"},
      {"file": "002_add_columns.sql", "line": 5, "operation": "ALTER", "source": "migration"},
      {"file": "003_user_updates.sql", "line": 45, "operation": "ALTER", "source": "migration"}
    ]
  }
]
```

## Workflow Example (PostgreSQL)

1. Navigate to PostgreSQL directory:
   ```bash
   cd postgresql
   ```

2. Set up environment configuration:
   ```bash
   cp debee.env.example debee.env
   # Edit debee.env with your database settings
   ```

3. Analyze existing migrations:
   ```bash
   python extract-db-objects.py --format markdown --output DB_SCHEMA.md
   ```

4. Run database migrations:
   ```powershell
   # Full service - recreate, restore, and update
   .\debee.ps1 -Operations fullService

   # Or specific operations
   .\debee.ps1 -Operations @("recreateDatabase", "updateDatabase")
   ```

5. Track changes after new migrations:
   ```bash
   python extract-db-objects.py --format json --output db_objects_v2.json
   ```

## Requirements

### For debee.ps1
- PowerShell 5.1 or later
- PostgreSQL client tools (psql, pg_restore)
- Access to PostgreSQL database

### For extract-db-objects.py
- Python 3.6 or later
- No external dependencies (uses standard library only)

## Example PostgreSQL Project Structure

```
my-project/
├── debee/
│   └── version_table_template.html # HTML template for interactive reports
├── debee.env                    # Environment configuration
├── .debee.env                   # Local overrides (git-ignored)
├── debee.py                     # Orchestrator script (copied from postgresql/)
├── extract-db-objects.py       # Object extraction script (copied from postgresql/)
├── 001_init.sql                # Migration files
├── 002_tables.sql
├── 003_functions.sql
├── scripts/
│   └── recreate_database.sql   # Database recreation logic
├── backups/
│   └── production.dump          # Database backups
├── hooks/
│   ├── pre_update.sql          # Pre-update scripts
│   └── post_update.sql         # Post-update scripts
└── ad-hoc-scripts/             # Emergency/hotfix scripts
    ├── hotfix_001_user_bug.sql
    ├── hotfix_002_data_fix.sql
    └── emergency_function_update.sql
```

## Best Practices

1. **Version Control**: Keep migration files in version control
2. **Naming Convention**: Use consistent 3-digit prefixes for migration files
3. **Environment Files**: Use `.debee.env` for local overrides (add to .gitignore)
4. **Documentation**: Run extract-db-objects.py regularly to document schema changes
5. **Testing**: Test migrations in development before production
6. **Backup**: Always backup before running destructive operations
7. **Ad-hoc Scripts**: Use descriptive names for emergency/hotfix scripts (e.g., `hotfix_001_fix_user_permissions.sql`)
8. **Ad-hoc Tracking**: Include ad-hoc directory in version control but keep it separate from migration files

## Security Notes

- Store sensitive credentials in `.debee.env` files
- Add `.debee*.env` to `.gitignore`
- Use environment-specific configuration files
- Restrict file permissions on configuration files

## Future Database Support

The project structure is designed to support additional databases:
```
debee/
├── postgresql/     # Current
├── mysql/          # Planned
├── sqlserver/      # Planned
└── oracle/         # Planned
```

## License

[Specify your license here]

## Contributing

[Add contributing guidelines if applicable]