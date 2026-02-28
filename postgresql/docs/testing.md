# Testing

Debee includes a test runner (`runTests` operation) for executing SQL-based database tests. Tests can be organized as flat files or structured suites with isolation modes and shared setup.

## Running Tests

```bash
# Run all tests
./debee.sh -o runTests
python debee.py -o runTests
.\debee.ps1 -Operations runTests

# Filter tests by name
./debee.sh -o runTests --test-filter connectivity
python debee.py -o runTests --test-filter connectivity
.\debee.ps1 -Operations runTests -TestFilter connectivity
```

## Test Organization

Tests live in a `tests/` directory and come in two forms:

### Flat Test Files

Single SQL files named `test_*.sql`:

```
tests/
├── test_connection.sql
├── test_permissions.sql
└── test_data_integrity.sql
```

### Suite Directories

Directories named `test_*/` containing numbered SQL files:

```
tests/
└── test_user_permissions/
    ├── test.json              # Optional manifest
    ├── 001_create_test_user.sql   # Setup
    ├── 002_grant_permissions.sql  # Setup
    ├── 003_verify_access.sql      # Verification
    └── 900_cleanup.sql            # Cleanup phase
```

Both forms are discovered automatically and can coexist in the same `tests/` directory.

## Suite File Phases

Files within a suite are organized by their numeric prefix:

| Prefix Range | Phase | Behavior |
|-------------|-------|----------|
| **000–899** | Main | Setup + execution + verification. Runs in order. Stops on first error or FAIL. |
| **900–999** | Cleanup | Always runs if `always_cleanup` is true. Failures logged as warnings. |

## Suite Manifest (`test.json`)

An optional `test.json` file in a suite directory controls suite behavior:

```json
{
  "name": "User Permissions",
  "description": "Verify role-based access controls",
  "always_cleanup": true,
  "isolation": "transaction",
  "setup": ["shared/create_schema.sql", "shared/seed_data.sql"]
}
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | string | Folder name humanized | Display name (e.g., `test_user_permissions` → `User Permissions`) |
| `description` | string | — | Shown in output header |
| `always_cleanup` | boolean | `true` | Whether cleanup phase (900–999) runs even on failure |
| `isolation` | string | `"none"` | Isolation mode: `"none"`, `"transaction"`, or `"database"` |
| `setup` | string[] | `[]` | Shared setup scripts, paths relative to `tests/` |

## Isolation Modes

### `"none"` (default)

No isolation. Each SQL file runs individually against the target database. Changes persist.

### `"transaction"`

Wraps all setup + main phase files in a single `BEGIN`/`ROLLBACK` psql session:

- All changes are rolled back automatically after the suite
- Uses `\set ON_ERROR_STOP on` to stop on SQL errors
- Per-file output attribution via `>>>DEBEE_FILE: ...<<<` markers
- Cleanup files (900–999) still run individually after the rollback
- A temporary wrapper SQL file is created and cleaned up automatically

```json
{
  "isolation": "transaction"
}
```

Best for: tests that should leave no trace in the database.

### `"database"`

Recreates (and optionally restores) the database before the suite:

- Calls `recreateDatabase` before suite execution
- Calls `restoreDatabase` if `DBBACKUPFILE` is configured
- Switches back to `DBDESTDB` for test execution
- Then runs shared setup + main files individually (same as `"none"`)

```json
{
  "isolation": "database"
}
```

Best for: tests that need a clean database state and may make irreversible changes.

## Shared Setup Scripts

The `setup` field in `test.json` specifies SQL files that run before the suite's own files. Paths are relative to the `tests/` directory:

```json
{
  "setup": ["shared/create_schema.sql", "shared/seed_data.sql"]
}
```

A `tests/shared/` directory is the convention for storing shared setup scripts.

Shared setup works with all isolation modes:
- `"none"` — setup files run individually
- `"transaction"` — setup files are included in the `BEGIN`/`ROLLBACK` wrapper
- `"database"` — setup files run individually after database recreation

## Global Test Ordering

Create `tests/tests.json` to control execution order:

```json
{
  "order": [
    "test_connection.sql",
    "test_connectivity"
  ]
}
```

- Items listed in `order` run first, in the specified order
- Unlisted items follow alphabetically after ordered items
- If `tests.json` is absent, all tests run alphabetically
- Ordering is applied before `--test-filter` filtering

## PASS/FAIL Detection

The test runner checks two things for each test file:

1. **psql exit code** — a nonzero exit code counts as automatic FAIL
2. **Output content** — the runner scans psql output for `FAIL` strings

Output is colorized: PASS results in green, FAIL results in red.

## Test Filtering

Use `--test-filter` to run a subset of tests by name:

```bash
# Run only tests matching "connectivity"
./debee.sh -o runTests --test-filter connectivity

# Run only tests matching "connection"
python debee.py -o runTests --test-filter connection
```

The filter matches against test file/directory names.
