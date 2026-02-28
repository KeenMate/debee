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
