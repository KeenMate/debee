# Cross-Platform Support

Debee provides three independent orchestrator implementations with identical functionality. Choose the one that fits your environment.

## Implementations

| Orchestrator | Language | Best For |
|-------------|----------|----------|
| `debee.ps1` | PowerShell | Windows environments, teams already using PowerShell |
| `debee.sh` | Bash | Linux/macOS native environments, CI/CD pipelines |
| `debee.py` | Python | Cross-platform consistency, teams already using Python |

All three implementations support every operation and produce identical results.

## Which to Choose

- **Windows** → `debee.ps1` (native) or `debee.py`
- **Linux/macOS** → `debee.sh` (native) or `debee.py`
- **Mixed environments** → `debee.py` for consistency
- **CI/CD pipelines** → `debee.sh` (lightweight) or `debee.py`

## Platform Compatibility

| Orchestrator | Windows | Linux | macOS | WSL |
|-------------|---------|-------|-------|-----|
| `debee.ps1` | Native | PowerShell Core | PowerShell Core | PowerShell Core |
| `debee.sh` | WSL only | Native | Native | Native |
| `debee.py` | Native | Native | Native | Native |

### Runtime Requirements

| Orchestrator | Runtime |
|-------------|---------|
| `debee.ps1` | PowerShell 5.1+ (Windows) or PowerShell Core 6.0+ |
| `debee.sh` | Bash 4.0+ |
| `debee.py` | Python 3.6+ |

All implementations also require PostgreSQL client tools (`psql`, `pg_restore`) and Python 3.6+ (for `extract-db-objects.py` used by `prepareVersionTable`).

## CLI Reference

### PowerShell (`debee.ps1`)

```powershell
.\debee.ps1
  -Operations <string[]>        # Operations to run (default: fullService)
  [-Environment <string>]       # Environment name for config file selection
  [-UpdateStartNumber <int>]    # First migration number (default: -1 = all)
  [-UpdateEndNumber <int>]      # Last migration number (default: -1 = all)
  [-SqlFile <string>]           # SQL file for execSql operation
  [-Sql <string>]               # Inline SQL for execSql operation
  [-TestFilter <string>]        # Test name filter for runTests (default: "all")
```

**Examples:**

```powershell
.\debee.ps1 -Operations fullService
.\debee.ps1 -Operations updateDatabase -UpdateStartNumber 10 -UpdateEndNumber 20
.\debee.ps1 -Operations @("restoreDatabase", "updateDatabase")
.\debee.ps1 -Operations execSql -SqlFile path/to/script.sql
.\debee.ps1 -Operations runTests -TestFilter connectivity
.\debee.ps1 -Environment staging -Operations updateDatabase
```

### Bash (`debee.sh`)

```bash
./debee.sh [options]
  -o, --operations <ops>      # Comma-separated operations (default: fullService)
  -e, --environment <env>     # Environment name for config file selection
  -s, --start-number <num>    # First migration number (default: -1 = all)
  -n, --end-number <num>      # Last migration number (default: -1 = all)
      --sql-file <file>       # SQL file for execSql operation
      --sql <query>           # Inline SQL for execSql operation
      --test-filter <pattern> # Test name filter for runTests (default: "all")
  -h, --help                  # Show help message
```

**Examples:**

```bash
./debee.sh -o fullService
./debee.sh -o updateDatabase -s 10 -n 20
./debee.sh -o restoreDatabase,updateDatabase
./debee.sh -o execSql --sql-file path/to/script.sql
./debee.sh -o runTests --test-filter connectivity
./debee.sh -e staging -o updateDatabase
```

### Python (`debee.py`)

```bash
python debee.py [options]
  -o, --operations <ops>      # Comma-separated operations (default: fullService)
  -e, --environment <env>     # Environment name for config file selection
  -s, --start-number <num>    # First migration number (default: -1 = all)
  -n, --end-number <num>      # Last migration number (default: -1 = all)
      --sql-file <file>       # SQL file for execSql operation
      --sql <query>           # Inline SQL for execSql operation
      --test-filter <pattern> # Test name filter for runTests (default: "all")
      --no-color              # Disable colored output (for CI/CD)
```

**Examples:**

```bash
python debee.py -o fullService
python debee.py -o updateDatabase -s 10 -n 20
python debee.py -o restoreDatabase,updateDatabase
python debee.py -o execSql --sql-file path/to/script.sql
python debee.py -o runTests --test-filter connectivity
python debee.py -e staging -o updateDatabase
python debee.py -o updateDatabase --no-color
```

## Valid Operations

All three implementations support the same set of operations:

- `recreateDatabase`
- `restoreDatabase`
- `updateDatabase`
- `preUpdateScripts`
- `postUpdateScripts`
- `prepareVersionTable`
- `execSql`
- `runTests`
- `fullService`

## Implementation Differences

| Feature | ps1 | sh | py |
|---------|-----|----|----|
| Multiple operations syntax | Array: `@("a", "b")` | Comma-separated: `a,b` | Comma-separated: `a,b` |
| Color output | PowerShell colors | ANSI escape codes | ANSI with TTY detection |
| Disable colors | — | — | `--no-color` flag |
| JSON parsing (test manifests) | ConvertFrom-Json | Python one-liner | json module |
| Environment inheritance | Automatic | Automatic | Explicit `env=os.environ` |

## Windows: UTF-8 Setup

If you pass SQL with non-ASCII characters via `--sql` / `-Sql` (e.g. Czech, German, Polish text) on Windows, you may see psql errors like:

```
ERROR:  invalid byte sequence for encoding "UTF8": 0xed 0x6b 0x6f
```

This happens because the Windows console / shell encodes the command-line string in the legacy ANSI code page (e.g. CP1250) instead of UTF-8, and psql rejects the resulting bytes. Debee passes `--sql` strings straight through to psql, so the fix belongs in your shell environment, not the orchestrator.

Configure your shell **once** and the problem goes away for all three orchestrators.

### PowerShell

Add to your `$PROFILE`:

```powershell
[Console]::InputEncoding  = [System.Text.UTF8Encoding]::new()
[Console]::OutputEncoding = [System.Text.UTF8Encoding]::new()
$OutputEncoding = [System.Text.UTF8Encoding]::new()
$env:PGCLIENTENCODING = 'UTF8'
chcp 65001 > $null
```

### Git Bash / MSYS

Add to `~/.bashrc`:

```bash
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
export PGCLIENTENCODING=UTF8
```

Also set the terminal itself to UTF-8: **mintty → right-click title bar → Options → Text → Character set: UTF-8**. Without this, typed characters become CP1250 bytes before bash ever sees them.

### Python

Set as system or per-shell environment variables:

```
PYTHONUTF8=1
PYTHONIOENCODING=utf-8
PGCLIENTENCODING=UTF8
```

`PYTHONUTF8=1` (Py 3.7+) forces Python's "UTF-8 mode" for all I/O and subprocess calls on Windows.

### Per-invocation fallback

If you can't change your shell config, set `PGCLIENTENCODING` to match whatever encoding the shell is actually sending — for Czech Windows that's usually `WIN1250`:

```bash
PGCLIENTENCODING=WIN1250 ./debee.sh -o execSql --sql "SELECT 'krejčíková';"
```

PostgreSQL will transcode the bytes server-side. Alternatively, save the SQL to a UTF-8 file and use `--sql-file` instead of `--sql`.

### System-wide option

Windows 10 1903+ has a UTF-8 system locale toggle: **Settings → Time & Language → Language → Administrative language settings → Change system locale → "Beta: Use Unicode UTF-8 for worldwide language support"**. Enabling it sets the system ANSI code page to UTF-8 (65001) and makes the per-shell config unnecessary. Some legacy apps that hardcode CP1250 assumptions can misbehave with it on.
