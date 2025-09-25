# Changelog - PostgreSQL Tools

All notable changes to the PostgreSQL database migration tools will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2024-01-25

### Added

#### Core Orchestrator (`debee.ps1`)
- **Migration Orchestration Engine**: PowerShell-based orchestrator for PostgreSQL operations
- **Environment Configuration System**: Support for `.env` files with environment-specific overrides
- **Operation Modes**:
  - `recreateDatabase`: Drop and recreate database from script
  - `restoreDatabase`: Restore from backup (file/directory/custom formats)
  - `updateDatabase`: Apply numbered migration files
  - `preUpdateScripts`: Execute pre-migration scripts
  - `postUpdateScripts`: Execute post-migration scripts
  - `fullService`: Run all operations in sequence
- **Migration File Processing**:
  - Numeric prefix pattern support (`XXX_description.sql`)
  - Range-based execution (`UpdateStartNumber`/`UpdateEndNumber`)
  - Automatic ordering of migration files
  - Special pattern support for `9X_` files and `999-examples.sql`
- **Backup Support**:
  - Multiple backup formats (file, directory, custom archive)
  - Configurable restore job count for parallel processing
  - Optional database creation during restore
- **Environment Variables**:
  - PostgreSQL connection settings (PGHOST, PGPORT, PGUSER, PGPASSWORD)
  - Database names configuration (DBCONNECTDB, DBDESTDB)
  - Tool paths configuration (DBPSQLFILE, DBPGRESTOREFILE)
  - Script paths for custom operations
- **Error Handling**:
  - Validation of operation parameters
  - File existence checks
  - Type safety for user prompts
  - Error stop mode for SQL execution (`-b` flag)

#### Database Object Extractor (`extract-db-objects.py`)
- **Object Detection** for PostgreSQL database objects:
  - Tables (including UNLOGGED and TEMPORARY)
  - Functions and Procedures
  - Indexes (including UNIQUE and CONCURRENT operations)
  - Views
  - Triggers
  - Schemas
- **Operation Tracking**:
  - CREATE operations
  - CREATE OR REPLACE operations
  - ALTER operations
  - DROP operations (with IF EXISTS support)
- **Change History**: Complete tracking of all modifications to each object
- **Multiple Output Formats**:
  - JSON with full object details and history
  - CSV for spreadsheet analysis
  - Markdown for documentation
- **Schema Support**: Full support for schema-qualified object names
- **File Pattern Recognition**:
  - Three-digit prefix pattern (`001_`, `002_`, etc.)
  - Special 90s pattern (`91_`, `92_`, etc.)
  - Examples file support (`999-examples.sql`)
- **Encoding Support**: Automatic fallback from UTF-8 to Latin-1
- **Summary Statistics**: Object counts by type and schema

#### Documentation
- **DEBEE_ORCHESTRATOR.md**: Comprehensive guide explaining orchestration architecture
- **README.md**: Main documentation with examples and configuration reference
- **CHANGELOG.md**: This file, tracking version history

### Architecture Decisions
- **Pure Orchestration Model**: No embedded SQL in orchestrator - all database logic in external files
- **Configuration-Driven**: All behavior controlled through environment variables
- **Stateless Execution**: Each run is independent, no state tracking between executions
- **Tool Agnostic**: Works with any PostgreSQL installation via psql/pg_restore
- **Cross-Platform**: PowerShell Core compatible for Windows/Linux/macOS

### Dependencies
- PowerShell 5.1+ (Windows) or PowerShell Core 6.0+ (Cross-platform)
- PostgreSQL client tools (psql, pg_restore)
- Python 3.6+ (for extract-db-objects.py)
- No external Python packages required (standard library only)

## [2.0.0] - 2024-01-25

### Added

#### Cross-Platform Orchestrators
- **debee.sh**: Complete Bash implementation of the orchestrator
  - Full feature parity with PowerShell version
  - Native Linux/macOS support
  - Color-coded output for better readability
  - Command-line argument parsing with getopt-style options
  - POSIX-compliant for maximum compatibility

- **debee.py**: Complete Python implementation of the orchestrator
  - Object-oriented architecture with `DebeeOrchestrator` class
  - Type hints for better code maintainability
  - Enum-based operation definitions
  - Enhanced error handling and subprocess management
  - Cross-platform compatibility (Windows/Linux/macOS)
  - Optional colored output with automatic TTY detection
  - `--no-color` flag for CI/CD environments

#### Common Features (Both New Orchestrators)
- **Consistent Interface**: Same command-line options across all implementations
- **Environment File Loading**: Support for main and local override files
- **Operation Modes**: All six operations from v1.0.0
- **Migration Range Selection**: Start/end number filtering
- **Error Propagation**: Proper exit codes for scripting
- **Verbose Logging**: Detailed operation progress reporting

### Changed
- **Project Structure**: Scripts now organized by database type (postgresql/)
- **Documentation**: Separated orchestrator documentation from main README

### Improved
- **Error Handling**: Better error messages and validation across all orchestrators
- **Platform Support**: Native implementations for each major platform
- **Code Maintainability**: Three independent implementations allow choosing based on environment

### Technical Details

#### debee.sh Implementation
- Uses bash built-in features for performance
- Array handling for file lists and operations
- Proper quote handling for paths with spaces
- Signal-safe script execution with `set -e`

#### debee.py Implementation
- Subprocess module for external command execution
- Path objects for file system operations
- Configurable output formatting
- Class-based design for extensibility

### Compatibility
| Orchestrator | Platform | Required Runtime |
|-------------|----------|------------------|
| debee.ps1   | Windows, Linux*, macOS* | PowerShell 5.1+ or PowerShell Core 6.0+ |
| debee.sh    | Linux, macOS, WSL | Bash 4.0+ |
| debee.py    | Windows, Linux, macOS | Python 3.6+ |

*With PowerShell Core installed

### Migration from v1.0.0
No breaking changes. All v1.0.0 configurations and scripts work with v2.0.0 orchestrators.
Choose the orchestrator that best fits your environment:
- Windows environments: Use `debee.ps1` (native) or `debee.py`
- Linux/macOS environments: Use `debee.sh` (native) or `debee.py`
- Mixed environments: Use `debee.py` for consistency

## [2.1.0] - 2025-01-25

### Added

#### Version Table Generation (`prepareVersionTable` operation)
- **New Operation**: `prepareVersionTable` available in all orchestrators (debee.ps1, debee.sh, debee.py)
- **Automated Documentation**: Generates comprehensive database object tracking tables
- **Dual Format Output**:
  - `db-objects.json`: Machine-readable format for further processing
  - `db-objects.md`: Human-readable markdown table for documentation
- **Complete Integration**: Leverages existing `extract-db-objects.py` script
- **Cross-Platform**: Consistent functionality across PowerShell, Bash, and Python implementations

#### Features
- **Object Extraction**: Automatically scans all migration files to identify database objects
- **Change Tracking**: Shows complete history of modifications for each object
- **Summary Statistics**: Provides counts by object type and schema
- **Markdown Table**: Generates formatted table matching postgresql-permissions-model format:
  - Schema and object name columns
  - Object type identification
  - Last update file and line number
  - Total update count
  - Complete update history with file references

#### Usage Examples
```bash
# Generate version table documentation
./debee.sh -o prepareVersionTable
./debee.ps1 -Operations prepareVersionTable
python debee.py -o prepareVersionTable
```

#### Output Files
- `db-objects.json`: Structured data with complete object history
- `db-objects.md`: Formatted markdown table for documentation

### Technical Implementation
- **Error Handling**: Validates Python availability and script existence
- **Python Detection**: Automatically detects `python3` or `python` commands
- **File Validation**: Ensures successful generation of both output formats
- **Consistent Interface**: Same operation name and behavior across all orchestrators

## [2.2.0] - 2025-01-25

### Added

#### Ad-hoc Scripts Support
- **New Configuration Variable**: `DBADHOCDIRECTORY` environment variable
  - Specifies directory containing emergency/hotfix scripts
  - Supports any SQL files for immediate database fixes
  - Scanned recursively for comprehensive coverage

#### Enhanced Object Extraction
- **Ad-hoc Integration**: `extract-db-objects.py` now scans ad-hoc scripts directory
- **Source Column**: Separate "Source" column distinguishes migration vs ad-hoc files for easy sorting
- **Clean Output**: No prefixes - clean file paths with dedicated column for source type
- **Comprehensive Tracking**: Objects modified by ad-hoc scripts tracked alongside migration changes
- **Update Classification**: Summary shows separate counts for migration vs ad-hoc updates

#### Configuration Integration
- **Environment Support**: All orchestrators recognize `DBADHOCDIRECTORY` configuration
- **Documentation**: Comprehensive examples and best practices for ad-hoc script management
- **Project Structure**: Updated recommended directory layout including ad-hoc scripts

### Use Cases
- **Emergency Fixes**: Quickly apply critical database patches
- **Hotfix Deployment**: Track emergency changes separate from planned migrations
- **Production Repairs**: Document urgent fixes that bypass normal migration process
- **Data Corrections**: Apply immediate data fixes while maintaining change history

### Features
- **Automatic Discovery**: Recursively scans configured directory for SQL files
- **Change Tracking**: Full history of what changed, when, and in which file
- **Source Classification**: Dedicated "Source" column shows "Migration" or "Ad-hoc"
- **Clean File Paths**: No prefixes clutter file names - source type in separate column
- **Sortable Output**: Easy to filter and sort by source type in all formats
- **Flexible Structure**: No naming conventions required - any SQL file is processed
- **Version Control**: Ad-hoc changes tracked in documentation like regular migrations

### Example Usage
```bash
# Set ad-hoc directory and generate documentation
DBADHOCDIRECTORY=hotfix-scripts/ python extract-db-objects.py --format markdown

# Shows output like:
# | auth | fix_user_permissions | function | hotfix-scripts/user_fix.sql | 15 | 1 | Ad-hoc | hotfix-scripts/user_fix.sql:15 |
```

### Migration from v2.1.0
Simply add `DBADHOCDIRECTORY=your-adhoc-directory/` to your environment configuration to enable the feature.

## [2.2.1] - 2025-01-25

### Fixed

#### Environment Variable Inheritance Issue
- **debee.py**: Fixed subprocess calls in `prepare_version_table()` operation to properly pass environment variables
  - Added `env=os.environ` parameter to both `extract-db-objects.py` subprocess calls
  - Ensures `DBADHOCDIRECTORY` and other environment variables are available to the Python subprocess
  - Resolves issue where ad-hoc scripts were not being detected when using Python orchestrator

#### Impact
- **debee.sh**: No changes needed - inherits environment variables automatically from parent shell
- **debee.ps1**: No changes needed - PowerShell handles environment variable inheritance correctly
- **debee.py**: Fixed to match behavior of other orchestrators

### Technical Details
The Python `subprocess.run()` calls were missing the `env` parameter, causing child processes to not receive environment variables set by the orchestrator. This specifically affected:
- Ad-hoc script detection via `DBADHOCDIRECTORY` environment variable
- Any other custom environment variables used by `extract-db-objects.py`

### Migration from v2.2.0
No configuration changes required. Existing setups will automatically benefit from the fix.

## [Unreleased]

### Planned Features
- Migration history tracking table
- Dry-run mode for operations
- Rollback support for migrations
- Parallel migration execution
- Migration dependencies declaration
- Automatic backup before operations
- Migration validation and linting
- Support for stored procedure migrations
- Database comparison tools
- Migration generation from database changes

### Under Consideration
- Support for other databases (MySQL, SQL Server, Oracle)
- Web-based UI for migration management
- Integration with CI/CD pipelines
- Docker container support
- Cloud database support (RDS, Azure Database, Cloud SQL)
- Migration performance analytics
- Automated testing framework for migrations

---

## Version History Notes

### Versioning Strategy
- Major version (1.x.x): Breaking changes to configuration or command interface
- Minor version (x.1.x): New features that are backward compatible
- Patch version (x.x.1): Bug fixes and minor improvements

### Compatibility Matrix
| Tool Version | PostgreSQL | PowerShell | Python |
|--------------|------------|------------|---------|
| 1.0.0        | 9.6+       | 5.1+       | 3.6+    |

### Migration Path
Version 1.0.0 is the initial release. Future versions will include migration instructions if breaking changes are introduced.

### Support Policy
- Latest version: Full support
- Previous minor version: Security fixes only
- Older versions: Community support

---

*For bug reports and feature requests, please create an issue in the project repository.*