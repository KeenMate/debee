# Version Table

The `prepareVersionTable` operation generates comprehensive documentation of all database objects found across your migration and ad-hoc scripts.

## Overview

Debee scans your SQL files, extracts database objects (tables, functions, views, indexes, triggers, schemas), and produces a tracking report showing every object's complete change history.

```bash
# Generate version table
./debee.sh -o prepareVersionTable
python debee.py -o prepareVersionTable
.\debee.ps1 -Operations prepareVersionTable
```

## Output Formats

Configure which formats to generate via the `DBVERSIONTABLEFORMATS` environment variable (semicolon-separated):

```bash
DBVERSIONTABLEFORMATS=html;json;md;csv
```

| Format | Output File | Description |
|--------|-------------|-------------|
| `json` | `db-objects.json` | Machine-readable, full object details and history |
| `md` | `db-objects.md` | Markdown table for documentation |
| `csv` | `db-objects.csv` | Spreadsheet-compatible format |
| `html` | `db-objects.html` | Interactive dashboard with filtering |

## Configuration

| Variable | Purpose | Example |
|----------|---------|---------|
| `DBVERSIONTABLEFORMATS` | Output formats to generate | `html;json;md` |
| `DBADHOCDIRECTORY` | Ad-hoc scripts to include in tracking | `ad-hoc-scripts/` |

## What Gets Tracked

### Object Types
- Tables (including UNLOGGED and TEMPORARY)
- Functions and Procedures
- Indexes (including UNIQUE and CONCURRENT)
- Views
- Triggers
- Schemas

### Operations
- CREATE
- CREATE OR REPLACE
- ALTER
- DROP (with IF EXISTS support)

### Per-Object Information
- Schema and object name
- Object type
- Last update file and line number
- Last operation type
- Total update count
- Complete update history with file references
- Source classification (Migration vs Ad-hoc)

## extract-db-objects.py

The `extract-db-objects.py` script does the actual SQL parsing. It:

- Scans all migration files following the `XXX_*.sql` naming pattern
- Scans the ad-hoc scripts directory (if `DBADHOCDIRECTORY` is set)
- Supports schema-qualified object names
- Handles encoding fallback (UTF-8 → Latin-1)
- Produces summary statistics by object type and schema

The orchestrators call this script automatically during `prepareVersionTable`.

### Direct Usage

```bash
# Generate specific format
python extract-db-objects.py --format markdown
python extract-db-objects.py --format json --output db-objects.json

# With ad-hoc scripts
DBADHOCDIRECTORY=hotfix-scripts/ python extract-db-objects.py --format html
```

## HTML Interactive Dashboard

The HTML output provides a full interactive dashboard for exploring database objects:

- **Real-time filtering** — 8 independent filter controls (schema, name, type, operation, file, migrations, ad-hoc, update count)
- **Click-to-filter** — click any schema, object name, or file to instantly filter
- **Responsive design** — desktop table view, mobile card layout
- **Local storage** — layout preferences persist across sessions
- **Conditional ad-hoc columns** — automatically hidden when no ad-hoc data exists

The dashboard is a single self-contained HTML file with embedded CSS and JavaScript — no external dependencies.

### Template

The HTML output is generated from `version_table_template.html`. The template uses data placeholders that are filled during generation.

## Ad-Hoc Script Tracking

When `DBADHOCDIRECTORY` is configured, objects modified by ad-hoc scripts are tracked alongside migration changes:

- A dedicated "Source" column distinguishes "Migration" vs "Ad-hoc" entries
- Separate update counts for migration and ad-hoc changes
- All formats support source classification
- No naming convention required for ad-hoc files — any `.sql` file is processed

```
# Example output row
| auth | fix_permissions | function | hotfix.sql | 15 | 1 | Ad-hoc | hotfix.sql:15 |
```
