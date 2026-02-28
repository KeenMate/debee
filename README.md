# Debee — Database Migration and Management Tools

A collection of database migration orchestration tools. Debee coordinates PostgreSQL operations — migrations, backups, testing, and documentation — without embedding any SQL itself. Your database logic stays in SQL files where it belongs.

## Database Support

### PostgreSQL

Complete toolset with three cross-platform orchestrator implementations (PowerShell, Bash, Python).

See the [PostgreSQL README](postgresql/README.md) for features, quick start, and architecture overview.

#### Documentation

| Topic | Description |
|-------|-------------|
| [Configuration](postgresql/docs/configuration.md) | Environment files, variables reference, local overrides |
| [Operations](postgresql/docs/operations.md) | All operations explained, migration naming, ad-hoc scripts |
| [Testing](postgresql/docs/testing.md) | Test runner, suites, isolation modes, shared setup |
| [Version Table](postgresql/docs/version-table.md) | Database object tracking, output formats, HTML dashboard |
| [Cross-Platform](postgresql/docs/cross-platform.md) | CLI reference for ps1/sh/py, platform guidance |
| [Changelog](postgresql/CHANGELOG.md) | Version history and migration notes |

#### Quick Usage

```bash
# PowerShell
.\debee.ps1 -Operations fullService

# Bash
./debee.sh -o fullService

# Python
python debee.py -o fullService
```

#### HTML Dashboard

![vivaldi_KLUI6zddbB](https://github.com/user-attachments/assets/028c96c8-7e3d-4f00-a3fc-bf4ebc78dbe7)

Interactive dashboard for exploring database objects across migrations. See [Version Table docs](postgresql/docs/version-table.md).

## Project Structure

```
debee/
├── README.md                              # This file
└── postgresql/                            # PostgreSQL tools
    ├── README.md                          # PostgreSQL overview & quick start
    ├── CHANGELOG.md                       # Version history
    ├── debee.ps1                          # PowerShell orchestrator
    ├── debee.sh                           # Bash orchestrator
    ├── debee.py                           # Python orchestrator
    ├── extract-db-objects.py              # Database object extractor
    ├── version_table_template.html        # HTML dashboard template
    └── docs/
        ├── configuration.md              # Environment & variables
        ├── operations.md                 # Operations reference
        ├── testing.md                    # Test framework
        ├── version-table.md              # Object tracking & formats
        └── cross-platform.md            # CLI reference & platform guide
```

## Future Database Support

The project structure is designed for expansion:

```
debee/
├── postgresql/     # Current
├── mysql/          # Planned
├── sqlserver/      # Planned
└── oracle/         # Planned
```

## Requirements

- PostgreSQL client tools (`psql`, `pg_restore`)
- One of: PowerShell 5.1+, Bash 4.0+, or Python 3.6+
- Python 3.6+ (for `extract-db-objects.py`, standard library only)

## Security Notes

- Store credentials in `.debee.env` files (gitignored)
- Use environment-specific configuration files
- Restrict file permissions on configuration files
