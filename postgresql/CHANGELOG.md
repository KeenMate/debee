# Changelog - PostgreSQL Tools

All notable changes to the PostgreSQL database migration tools will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-08-03

### Added
- **`--llm` reference flag**: New `--llm` flag (bash, Python) and `-Llm` switch (PowerShell) prints a single self-contained CLI reference document — concept, invocation for all three implementations, every operation, the `NNN_*.sql` naming convention, all options, the `.env` file resolution order, every environment variable, and the production-confirmation gate — designed to be pasted into an LLM/AI assistant as full context. Modeled on the `--llm` flag in the sibling `pure-admin-cli` project. The reference text is ASCII-only (no em-dashes or arrows) so it prints cleanly on Windows consoles regardless of code page, and is byte-for-byte identical across `debee.ps1`, `debee.sh`, and `debee.py`. Advertised in each script's short help. Version bumped to 1.1.0 across all three orchestrators.

## [1.0.5] - 2026-07-30

### Fixed
- **`restoreDatabase` broken in `debee.py` for custom/dir archives**: The pg_restore format flag was built as a single string `"-F c"` (or `"-F d"`) and passed as one element of the `subprocess.run` argv list. Because the list form bypasses the shell, no whitespace tokenization happens — pg_restore received a single argument `-F c` and parsed the format as `" c"`, aborting with `unrecognized archive format " c"; please specify "c", "d", or "t"`. Changed to the attached form `-Fc` / `-Fd` (a single valid token). Only `debee.py` was affected: `debee.sh` and `debee.ps1` already pass `-F` and the format letter as separate argv tokens. Version bumped in lockstep across all three orchestrators; no behavioral change to `debee.sh` / `debee.ps1`.

## [1.0.4] - 2026-06-19

### Fixed
- **Migration range resolution in production confirmation**: When `DBPRODENVIRONMENT=true` was set, the confirmation banner displayed `Migration range: -1 -> -1` even if `DBUPDATESTARTNUMBER` / `DBUPDATEENDNUMBER` were set in the env file. The fallback from CLI args to env vars was only resolved inside the file-gatherer, which ran *after* the confirmation. Resolution is now performed once right after env files load, so `Confirm-Production` and the file-gatherer share the same values. CLI args still take precedence over env vars. Fixed in `debee.ps1`, `debee.sh`, and `debee.py`.

### Changed
- **Friendlier range display**: Migration range output in the production confirmation banner and the "Scripts to run" warning now reads as `all` (both unset), `78 onwards` (only start set), `up to 100` (only end set), or `78 -> 100` (both set), instead of leaking the internal `-1` sentinels. Applies to all three orchestrators.

## [1.0.3] - 2026-05-31

### Fixed
- **PowerShell UTF-8 encoding**: `debee.ps1` now starts with a UTF-8 BOM (`EF BB BF`) so Windows PowerShell 5.1 decodes the script as UTF-8 instead of Windows-1252. Without the BOM, any non-ASCII content in the script (em-dashes, accented characters in help text or messages) was mangled at parse time, which surfaced as garbled console output and could break string comparisons. PowerShell 7+ already defaults to UTF-8 and is unaffected. `debee.sh` and `debee.py` versions bumped in lockstep — no behavioral change to those orchestrators.

## [1.0.2] - 2026-05-27

### Added
- **Silent mode**: New `-q`/`--silent` flag (bash, Python) and `-Silent`/`-q` switch (PowerShell) suppresses orchestration scaffolding output — env-file load messages, "Processing: <op>" / "Performing <op>..." banners, execSql banners, migration progress lines, and the final "All operations completed successfully!" line. Warnings, errors, and psql/pg_restore output remain visible, so piping `execSql --sql "..."` results no longer requires `2>/dev/null` tricks. Test runner output is unaffected (still controlled by `--test-verbose`).
- **Version flag**: New `-V`/`--version` flag (bash, Python) and `-Version`/`-V` switch (PowerShell) prints the orchestrator version and exits. Version constants (`DEBEE_VERSION` in bash, `__version__` in Python, `$DebeeVersion` in PowerShell) are now hard-coded at the top of each script so the CHANGELOG number is verifiable from the binary.

- **Production confirmation**: New env variable `DBPRODENVIRONMENT=true` marks an env file as production. When set, debee prints a summary (host, port, user, target DB, operations, SQL or migration range when applicable) and requires the user to type `yes` (full word, case-insensitive) before any operation runs. New `-y`/`--yes` flag (bash, Python) and `-Yes`/`-y` switch (PowerShell) bypasses the prompt for automation/CI use. The confirmation banner prints regardless of `--silent` — it is not orchestration chatter, it is a safety gate.

### Changed
- **PowerShell help screen**: `.\debee.ps1` (no args) previously triggered an interactive prompt for the mandatory `-Operations` parameter; it now prints a styled help screen listing operations, options, and examples. The `-Operations` parameter is no longer mandatory and has no default — running fullService is now an explicit choice (`-Operations fullService`) rather than the silent default of an empty invocation. New `-Help`/`-h`/`-?` switch shows the same screen explicitly.

### Docs
- **Windows UTF-8 setup**: New section in `docs/cross-platform.md` explaining how to mitigate `invalid byte sequence for encoding "UTF8"` errors when passing non-ASCII text via `--sql` on Windows. Covers per-shell configuration for PowerShell (`$PROFILE`), Git Bash (`~/.bashrc` + mintty), and Python (`PYTHONUTF8=1`), plus a `PGCLIENTENCODING=WIN1250` per-invocation fallback and the Windows system-wide UTF-8 locale toggle. No script changes — the fix lives in the shell environment, since debee passes `--sql` strings straight through to psql.

## [1.0.1] - 2026-03-05

### Fixed
- **Test Runner (transaction mode)**: SQL error messages (e.g., `ERROR: column does not exist`) are now shown in silent mode. Previously, only sections containing `FAIL` strings were displayed — psql errors without `FAIL` were silently swallowed, forcing users to rerun with `--test-verbose` or manual wrappers to diagnose failures. Fixed in all three orchestrators (`debee.ps1`, `debee.sh`, `debee.py`).

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

## [3.0.0] - 2025-01-27

### Added

#### HTML Interactive Dashboard
- **Complete HTML-based Version Table**: Interactive dashboard replacing markdown output
- **Professional UI Design**: Clean, Audi-inspired design with sophisticated styling
- **Real-time Filtering System**: 8 independent filter controls for comprehensive data exploration
  - Schema filter with text search
  - Object Name filter with text search
  - Object Type dropdown with all detected types
  - Operation filter for database operations (CREATE, ALTER, DROP, etc.)
  - Last File filter for finding specific migration files
  - Migration Updates filter for tracking specific migration scripts
  - Ad-hoc Updates filter for emergency script tracking
  - Minimum Update Count filter for finding frequently modified objects
- **Enhanced Data Display**: Rich information display with operation tracking
  - File:Line:Operation format showing exact change location and type
  - Complete migration and ad-hoc update history per object
  - Color-coded object type badges for quick identification
- **Advanced Interactions**: Clickable elements for intuitive filtering
  - Click any schema, object name, file name to instantly filter
  - Smart filename extraction from complex file:line:operation strings
  - Cross-column filtering for complex data exploration

#### Responsive Design System
- **Mobile-First Architecture**: Complete responsive design for all screen sizes
- **Card-Based Mobile Layout**: Touch-friendly card interface for mobile devices
  - Compact object cards showing all essential information
  - Optimized typography and spacing for mobile viewing
  - Natural page scrolling instead of constrained containers
- **Breakpoint System**: Professional responsive behavior
  - Desktop (1200px+): Full table with 4-column filter grid
  - Laptop (992px-1199px): Table with 3-column filter grid
  - Tablet (768px-991px): Table with 2-column filter grid
  - Mobile (<768px): Card layout with single-column filters
- **Adaptive Interface Elements**:
  - Header reorganization: Statistics hidden on mobile for space optimization
  - Icon-based controls using UTF-8 symbols for universal compatibility
  - Filter toggle: Show/hide filters on mobile to maximize content space
  - Layout toggle: Full-width mode removes all padding on mobile

#### Professional UI Components
- **Icon Button System**: Square 32x32px buttons with UTF-8 icons
  - Layout toggle: ◧ (contained) / ◨ (full width)
  - Filter toggle: ▲ (hide) / ▼ (show)
  - Hover states and tooltips for accessibility
- **Smart Statistics Display**: Dynamic positioning and visibility
  - Desktop: Column layout on right side of header
  - Mobile: Completely hidden to maximize content space
  - Real-time updates showing filtered vs total counts
- **Conditional Ad-hoc Support**: Intelligent feature detection
  - Automatically hides ad-hoc columns and filters when no ad-hoc data present
  - Dynamic column count and layout adjustment
  - Clean interface for projects not using ad-hoc scripts

#### Advanced Features
- **Local Storage Preferences**: Persistent user settings
  - Layout preference (contained/full-width) saved globally across all debee files
  - Automatic preference loading on page startup
  - Cross-session consistency for better user experience
- **Dynamic Height Management**: Optimized viewport utilization
  - Desktop filters visible: 55vh table height
  - Desktop filters hidden: 72vh table height for maximum data viewing
  - Mobile: Natural scrolling without height constraints
- **Enhanced Operation Tracking**: Complete database change history
  - Shows exact operation type (CREATE, CREATE_OR_REPLACE, ALTER, DROP)
  - File, line number, and operation in unified display format
  - Operation-based filtering for analyzing change patterns

### Enhanced

#### Extract DB Objects (`extract-db-objects.py`)
- **Operation Data Enhancement**: Added `last_update_operation` field to object tracking
- **HTML Template Integration**: Automatic template-based HTML generation
- **Ad-hoc Detection**: Smart detection of ad-hoc updates for conditional UI features
- **JavaScript Data Injection**: Dynamic data loading with operation information

#### Version Table Template (`version_table_template.html`)
- **Template-Based Generation**: Reusable template for consistent HTML output
- **Data Placeholder System**: Automatic data injection during generation
- **Cross-Platform Compatibility**: Works with all debee orchestrators

### Changed

#### User Interface Paradigm
- **From Static to Interactive**: Complete migration from markdown tables to interactive dashboard
- **Mobile-First Approach**: Responsive design prioritizing mobile user experience
- **Professional Aesthetics**: Modern, clean design suitable for enterprise environments

#### Data Presentation
- **Enhanced Information Density**: More data visible with better organization
- **Operation Visibility**: Database operations now prominently displayed
- **Smart Filtering**: Intuitive click-to-filter interaction model

### Technical Improvements

#### Performance Optimizations
- **Client-Side Processing**: All filtering and sorting happens in browser
- **Efficient Data Structure**: Optimized JavaScript data format
- **Minimal HTTP Requests**: Single-file solution with embedded assets

#### Code Organization
- **Modular JavaScript**: Well-structured code with clear separation of concerns
- **CSS Grid Layout**: Modern CSS for responsive design
- **Progressive Enhancement**: Works without JavaScript (basic table display)

### Accessibility Features
- **Keyboard Navigation**: Full keyboard support for all interactive elements
- **Screen Reader Support**: Proper ARIA labels and semantic HTML
- **High Contrast**: Professional color scheme with good contrast ratios
- **Touch-Friendly**: Appropriate touch targets for mobile devices

### Browser Compatibility
- **Modern Browsers**: Chrome, Firefox, Safari, Edge (latest versions)
- **Mobile Browsers**: iOS Safari, Chrome Mobile, Samsung Internet
- **Responsive Images**: Scales properly on high-DPI displays

### Migration from v2.2.1
- **Automatic HTML Generation**: Add `html` to `DBVERSIONTABLEFORMATS` environment variable
- **Template Inclusion**: Ensure `debee/version_table_template.html` is present in project
- **No Breaking Changes**: All existing markdown and JSON outputs continue to work
- **Enhanced Workflow**: HTML dashboard provides superior data exploration capabilities

### Example Usage
```bash
# Generate interactive HTML dashboard
DBVERSIONTABLEFORMATS="html;json;md" python extract-db-objects.py --format html --output db-objects.html

# Using orchestrators
./debee.sh -o prepareVersionTable  # Generates all configured formats including HTML
```

## [3.0.1] - 2025-11-26

### Fixed

#### File Filtering in Migration Processing
- **debee.ps1**: Fixed `Get-FilesByNumericPrefix` function to properly filter migration files
  - Added explicit validation for file pattern `^\d{3}_.*\.sql$`
  - Files must now start with exactly 3 digits, followed by underscore, and end with `.sql` extension
  - Prevents non-SQL files (like `ALL_TASKS_COMPLETE.md`) from being incorrectly processed
  - Adds informative skip message when files don't match expected pattern

#### Root Cause
The original filter `???_*` matched ANY 3 characters + underscore, not just digits. Files like `ALL_TASKS_COMPLETE.md` would pass the initial filter, then fail silently during numeric prefix extraction.

#### Impact
- Only affects `debee.ps1` orchestrator
- No configuration changes required
- Existing valid migration files (e.g., `060_create_table.sql`) continue to work normally

## [3.1.0] - 2026-02-28

### Added

#### Folder-Based Test Suite Framework (`runTests` operation)
- **Suite Directories**: Test suites can now be organized as `test_*/` directories alongside flat `test_*.sql` files
- **Optional Manifest**: `test.json` file in suite directories for metadata
  - `name`: Display name (default: folder name humanized, e.g. `test_user_permissions` → `User Permissions`)
  - `description`: Optional description shown in output header
  - `always_cleanup`: Controls whether cleanup phase runs on failure (default: `true`)
- **Phase Convention**: Files within suites are organized by numeric prefix
  - **000–899**: Main phase (setup + execution + verification), runs in order, stops on first error or FAIL
  - **900–999**: Cleanup phase, always runs if `always_cleanup` is true, failures logged as warnings
- **Suite Discovery**: Automatically discovers both flat `test_*.sql` files and `test_*/` directories in `tests/`
- **Enhanced Summary**: Per-suite and per-file result counts alongside PASS/FAIL totals
- **Sample Test Suite**: `test_connectivity/` suite with setup, verification, and cleanup phases

### Improved

#### Error Detection in Test Runner
- **psql Exit Code Handling**: Nonzero psql exit codes now count as automatic FAIL across all orchestrators
- Previously psql errors could be silently ignored if no FAIL string appeared in output

### Technical Details

#### New Helper Functions
| Orchestrator | Functions Added |
|-------------|----------------|
| debee.py | `_read_test_manifest()`, `_invoke_test_sql_file()`, `_invoke_flat_test()`, `_invoke_suite_test()` |
| debee.ps1 | `Read-TestManifest`, `Invoke-TestSqlFile`, `Invoke-FlatTest`, `Invoke-SuiteTest` |
| debee.sh | `read_test_manifest`, `invoke_test_sql_file`, `invoke_flat_test`, `invoke_suite_test` |

#### New Data Structures (debee.py)
- `TestManifest` dataclass: Holds suite manifest data with defaults
- `TestResult` dataclass: Structured test result with pass/fail counts and error flag

#### JSON Parsing (debee.sh)
- Uses `python3 -c` one-liner for JSON manifest parsing (Python already required by project)

### Migration from v3.0.1
- No breaking changes — existing flat `test_*.sql` files continue to work unchanged
- To use suites, create a `test_*/` directory inside `tests/` with numbered SQL files
- Optional `test.json` manifest for custom suite name/description

### Example Usage
```bash
# Run all tests (flat files + suites)
./debee.sh -o runTests
python debee.py -o runTests
.\debee.ps1 -Operations runTests

# Filter to run only suite tests
./debee.sh -o runTests --test-filter connectivity

# Filter to run only flat file tests
./debee.sh -o runTests --test-filter connection
```

## [3.2.0] - 2026-02-28

### Added

#### Test Suite Isolation Modes
- **Transaction Isolation** (`"isolation": "transaction"`): Wraps all setup + main files in a single `BEGIN`/`ROLLBACK` psql session
  - True transaction isolation — all changes are rolled back automatically
  - Uses `\set ON_ERROR_STOP on` to stop on SQL errors
  - Per-file output attribution via `>>>DEBEE_FILE: ...<<<` markers
  - Cleanup files (900-999) still run individually after the rollback
  - Temporary wrapper SQL file created and cleaned up in finally block
- **Database Isolation** (`"isolation": "database"`): Recreates and optionally restores the database before the suite
  - Calls existing `recreate_database()` before suite execution
  - Calls `restore_database()` if backup is configured (`DBBACKUPFILE`)
  - Switches back to `DBDESTDB` for test execution
  - Then runs shared setup + main files individually (same as `"none"`)
- **None Isolation** (`"isolation": "none"`): Current behavior, unchanged (default)
- Configured via `isolation` field in suite `test.json` manifest
- Unknown isolation values produce a warning and fall back to `"none"`

#### Shared Setup Scripts
- **`setup` field** in suite `test.json`: Array of paths relative to `tests/` directory
- Shared SQL files run before suite's own main files
- Enables reuse of common SQL (schema creation, seed data) across suites
- Works with all isolation modes:
  - `"none"`: Setup files run individually
  - `"transaction"`: Setup files included in the BEGIN/ROLLBACK wrapper
  - `"database"`: Setup files run individually after database recreation
- `tests/shared/` directory created as convention placeholder

#### Global Test Ordering
- **`tests/tests.json`**: Optional manifest for controlling test execution order
- `order` array specifies items to run first (in given order)
- Unlisted items follow alphabetically after ordered items
- If file is absent, current alphabetical behavior is preserved
- Works with `--test-filter` (ordering applied before filtering)

### Example Configuration

#### Suite `test.json` with isolation and shared setup:
```json
{
  "name": "Permissions Test",
  "description": "Test user permissions",
  "always_cleanup": true,
  "isolation": "transaction",
  "setup": ["shared/create_schema.sql", "shared/seed_data.sql"]
}
```

#### Global `tests/tests.json` with ordering:
```json
{
  "order": [
    "test_connection.sql",
    "test_connectivity"
  ]
}
```

### Technical Details

#### New Functions
| Orchestrator | Functions Added |
|-------------|----------------|
| debee.py | `_invoke_suite_transaction()` |
| debee.ps1 | `Invoke-SuiteTransaction` |
| debee.sh | `invoke_suite_transaction` |

#### Updated Data Structures
- `TestManifest` dataclass (debee.py): Added `isolation: str` and `setup: List[str]` fields
- Manifest hashtable (debee.ps1): Added `Isolation` and `Setup` keys
- Manifest globals (debee.sh): Added `_MANIFEST_ISOLATION` and `_MANIFEST_SETUP` variables

### Migration from v3.1.0
- No breaking changes — all existing test configurations work unchanged
- Add `"isolation": "transaction"` or `"isolation": "database"` to suite `test.json` to enable isolation
- Add `"setup": [...]` to suite `test.json` to enable shared setup scripts
- Create `tests/tests.json` with `"order": [...]` to control execution order

## [3.2.1] - 2026-02-28

### Added

#### Silent-by-Default Test Runner
- **`--test-verbose` flag**: All three orchestrators now default to **silent mode** when running tests
  - Silent mode only prints the summary block and any failures — PASS lines are suppressed
  - On failure, the relevant test/suite header and FAIL/ERROR lines are printed for context
  - Use `--test-verbose` (Bash, Python) or `-TestVerbose` (PowerShell) to restore full output
- **Token-efficient for AI tools**: Reduces output noise when tests pass, ideal for CI and AI-assisted workflows

#### Usage
```bash
# Silent mode (default) — only summary + failures
./debee.sh -o runTests
python debee.py -o runTests
.\debee.ps1 -Operations runTests

# Verbose mode — all PASS/FAIL lines shown (previous behavior)
./debee.sh -o runTests --test-verbose
python debee.py -o runTests --test-verbose
.\debee.ps1 -Operations runTests -TestVerbose
```

### Fixed

#### PowerShell Measure-Object Bug
- **debee.ps1**: Fixed `Measure-Object -Property PassCount -Sum` failing on hashtable results in test summary
  - Changed to `ForEach-Object { $_.PassCount } | Measure-Object -Sum` pattern
  - Same fix applied for `FailCount`
  - Root cause: `Measure-Object -Property` does not work on hashtable values, only on object properties

### Migration from v3.2.0
- No breaking changes — tests now produce less output by default
- Add `--test-verbose` or `-TestVerbose` to restore previous verbose behavior

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