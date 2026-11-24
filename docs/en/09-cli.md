# 9. CLI Reference


The CLI lives in the program directory (`esther-cli.exe` on Windows) and is suited to scripted batch analysis.

### 9.1 Analyzing SQL

```bash
# Analyze a SQL string, table output
esther-cli.exe analyze --dialect mysql --sql "SELECT a.id FROM orders a"

# Auto-detect the dialect, analyze a SQL file, JSON output (redirect to save)
esther-cli.exe analyze --dialect auto --file query.sql --format json > result.json

# Specify project and graph DB path
esther-cli.exe analyze -d postgres -s "SELECT * FROM t" --project etl --db-path esther.db
```

| Parameter | Description |
|-----------|-------------|
| `--dialect / -d` | Dialect ID (**required**); supports `auto` and all aliases |
| `--sql / -s` | SQL text |
| `--file / -f` | Path to a SQL file |
| `--format` | `json` or `table` (default) |
| `--project / -p` | Project name, default `default` |
| `--db-path` | Graph database file path |
| `--config / -c` | Config file path; defaults to `esther.toml` next to the executable |

### 9.2 Other commands

```bash
esther-cli.exe dialects list        # List all dialects and aliases
esther-cli.exe project create myprj # Create a project
esther-cli.exe project list         # List projects
esther-cli.exe project export myprj # Export all analyses in a project as JSON
esther-cli.exe license status       # Show license status
esther-cli.exe license fingerprint  # Print the machine fingerprint
```

### 9.3 Exit codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Success with parse warnings; also returned when `license status` detects a blocked license |
| `2` | Error (unknown dialect / missing file / bad format) |
| `3` | License blocked (unlicensed or expired) |

---
