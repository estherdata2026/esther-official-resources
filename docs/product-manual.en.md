# Esther Product Manual

**Esther · SQL Data Lineage Analysis Platform | The Wise Guardian of Your Data**

| | |
|------|------|
| Applies to | 0.10.0-preview (Windows preview build) |
| Last updated | 2026-08-23 |
| 中文版 | [product-manual.zh-CN.md](product-manual.zh-CN.md) |

---

## Contents

1. [Product Overview](#1-product-overview)
2. [Installation & Licensing](#2-installation--licensing)
3. [Quick Start: Your First Lineage Analysis](#3-quick-start-your-first-lineage-analysis)
4. [Data Lineage Analysis (Core Feature)](#4-data-lineage-analysis-core-feature)
   - [4.1 What column-level lineage is](#41-what-column-level-lineage-is)
   - [4.2 Three ways to run an analysis](#42-three-ways-to-run-an-analysis)
   - [4.3 Supported SQL dialects](#43-supported-sql-dialects)
   - [4.4 What SQL can be analyzed](#44-what-sql-can-be-analyzed)
   - [4.5 How to read the lineage graph](#45-how-to-read-the-lineage-graph)
   - [4.6 Graph interactions](#46-graph-interactions)
   - [4.7 Upstream tracing & downstream impact analysis](#47-upstream-tracing--downstream-impact-analysis)
   - [4.8 Metadata assistance: making lineage more accurate](#48-metadata-assistance-making-lineage-more-accurate)
5. [REST API Reference](#5-rest-api-reference)
6. [CLI Reference](#6-cli-reference)
7. [Configuration](#7-configuration)
8. [FAQ & Known Limitations](#8-faq--known-limitations)
9. [Getting Help](#9-getting-help)

---

## 1. Product Overview

**Esther** is an enterprise SQL data lineage analysis platform. It parses SQL text directly and extracts **column-level** data flow relationships — where a report field's data comes from, what computations happen along the way, and where it flows to — presented as an interactive lineage graph.

### 1.1 What problems it solves

| Scenario | Without lineage | With Esther |
|----------|-----------------|-------------|
| **Change impact analysis** | Changing a base column — no idea which downstream reports break | Select the column on the graph and see the entire downstream chain |
| **Data governance** | Relationships between tables/views/procedures buried in thousands of lines of SQL | Automatic parsing builds the full lineage graph |
| **Compliance / audit** | "Where does this external field come from?" answered by manual code reading | Upstream tracing walks back to data origins |
| **Migration / refactoring assessment** | Manual effort estimates that miss things | Quantify affected objects from the graph |

### 1.2 Three usage modes

- **Web UI** — paste SQL in the browser, get an interactive graph instantly
- **REST API** — full OpenAPI/Swagger documentation, easy to integrate
- **Command line CLI** — scripted batch analysis, JSON or table output

### 1.3 Capability highlights

- Column-level lineage with 9 edge types (direct / transform / aggregation / filter / join / …)
- 28 SQL dialects covering mainstream databases and Chinese domestic databases, with automatic detection
- Deep procedure parsing: PL/SQL, PL/pgSQL, T-SQL, MySQL procedures — variables, cursors, control flow, triggers
- Upstream tracing / downstream impact analysis backed by an embedded graph database
- Export lineage as PNG images, JSON, CSV

---

## 2. Installation & Licensing

### 2.1 Deploy in three steps (Windows preview build)

1. Download the zip package from [GitHub Releases](https://github.com/estherdata2026/esther-official-resources/releases) or [Gitee releases](https://gitee.com/esther2026/esther-official-resources/releases) and extract it anywhere on the target server (portable, no installer).
2. Double-click `start_esther.bat` to start the server.
3. Open `http://<server-ip>:8000` in your browser.

**Changing the port**: set the `ESTHER_PORT` environment variable and restart.

### 2.2 Licensing

- **Trial mode**: the first run starts with a built-in trial period; all features are available until it expires.
- **Activating a license**:
  1. Run `esther-cli.exe license fingerprint` to get the machine fingerprint;
  2. Email the fingerprint to [estherdata@163.com](mailto:estherdata@163.com) to receive the license file `esther.lic`;
  3. Place `esther.lic` in the program directory and restart — the server enters registered mode.
- **Checking status**: `esther-cli.exe license status`.

### 2.3 Runtime data & accounts

- Runtime data is created automatically under `data/` (SQLite + Kuzu graph database) — no external database required.
- The default administrator account is printed in the first startup log.
- Licenses are bound to the machine fingerprint; migrating servers requires a new license.

### 2.4 Security notes

- The program is a Nuitka native build; some antivirus products may false-positive on it — add it to the allowlist. Dependencies are statically linked; no VC++ runtime installation is required.
- `POST /api/analyze` currently has no authentication — deploy on an **internal network**. The `/api/v1/*` endpoints require a login token.

---

## 3. Quick Start: Your First Lineage Analysis

After opening the system in your browser:

1. Click **Data Lineage** (**数据血缘**) in the navigation bar. On first load the page **automatically runs a demo SQL** and renders the lineage graph — you see results immediately.
2. Paste your own SQL into the editor (multiple statements, stored procedures and CREATE statements are all fine).
3. Pick the dialect from the dropdown; leave **Auto-detect** if unsure.
4. Click **Analyze** (shortcut: `Ctrl + Enter`).
5. Inspect the results:
   - Stats bar: **node / edge / warning counts** and the detected dialect
   - **Lineage graph** tab: the interactive graph (see [4.5](#45-how-to-read-the-lineage-graph) and [4.6](#46-graph-interactions))
   - **JSON** tab: the raw result, identical to the API response
   - Warnings panel: click a warning to locate it in the SQL

> Note: the analysis page is an ad-hoc sandbox — results are not persisted. For persistent lineage that supports cross-object tracing, use metadata ingestion pipelines (see [4.7](#47-upstream-tracing--downstream-impact-analysis)).

---

## 4. Data Lineage Analysis (Core Feature)

### 4.1 What column-level lineage is

Given a piece of SQL, Esther extracts *which column flows into which column* and *what happens along the way*. For example:

```sql
INSERT INTO report (id, total)
SELECT order_id, price * qty
FROM orders;
```

Esther produces:

```text
┌──────────────┐                     ┌──────────────┐
│    orders    │                     │    report    │
│──────────────│                     │──────────────│
│  order_id    │───────DIRECT──────▶ │      id      │
│    price     │──┐                  │──────────────│
│     qty      │──┘──TRANSFORM ●𝑓──▶ │     total    │
│              │        price*qty    └──────────────┘
└──────────────┘
```

- `orders.order_id` passes **unchanged** into `report.id` → gray solid `DIRECT` edge
- `report.total` is **computed** from `price * qty` → purple dashed `TRANSFORM` edge; hover the `𝑓` badge to see the expression

Table-level lineage (table → table) is derived automatically by grouping columns.

### 4.2 Three ways to run an analysis

| | Web UI analysis page | REST API | CLI |
|---|---|---|---|
| **Entry** | "Data Lineage" nav item | `POST /api/analyze` | `esther-cli.exe analyze` |
| **Best for** | Interactive exploration | Integration, batch calls | Scripting, CI, offline analysis |
| **Input** | SQL text + dialect | SQL + dialect + optional schemas | SQL text or a `.sql` file |
| **Output** | Interactive graph + JSON | JSON (same as the UI's JSON tab) | JSON or terminal tables |
| **See** | Chapter 3 | Chapter 5 | Chapter 6 |

All three share the same parsing engine and produce identical result structures.

### 4.3 Supported SQL dialects

Esther supports **28** SQL dialects. Common aliases (e.g. `pg`, `mysql`) are accepted as dialect identifiers.

**International databases**

| Database | Dialect ID | Aliases |
|----------|-----------|---------|
| MySQL | `mysql` | `mariadb`, `tidb` |
| PostgreSQL | `postgres` | `postgresql`, `pg`, `opengauss` |
| Oracle | `oracle` | `oracledb` |
| SQL Server (T-SQL) | `tsql` | `sqlserver`, `mssql` |
| IBM DB2 | `db2` | |
| SAP HANA | `hana` | `saphana` |
| Sybase ASE | `sybase` | `ase` |
| Informix | `informix` | |
| Teradata | `teradata` | `td` |
| SQLite | `sqlite` | `sqlite3` |
| ClickHouse | `clickhouse` | `ch` |
| Snowflake | `snowflake` | |
| BigQuery | `bigquery` | |
| Presto | `presto` | `prestodb`, `prestosql` |
| Trino | `trino` | |
| Hive | `hive` | `hiveql` |
| Spark / Databricks | `spark` | `sparksql`, `databricks`, `delta` |
| Amazon Redshift | `redshift` | |
| Apache Doris | `doris` | `apachedoris` |
| StarRocks | `starrocks` | |
| DuckDB | `duckdb` | |

**Chinese domestic (Xinchuang) databases**

| Database | Dialect ID | Aliases |
|----------|-----------|---------|
| Dameng DM | `dameng` | `dm`, `dm7`, `dm8` |
| KingbaseES | `kingbase` | `kingbasees`, `kes` |
| ShenTong Oscar | `oscar` | `shentong` |
| HighGo | `highgo` | |
| GBase | `gbase` | `gbase8s`, `gbase8a` |
| OceanBase (MySQL mode) | `oceanbase` | `ob`, `oceanbase_mysql` |
| OceanBase (Oracle mode) | `oceanbase_oracle` | |
| GaussDB | `gaussdb` | |

**Auto-detection**: with "Auto-detect" (or `auto`), Esther first narrows candidates by keyword fingerprints, then parses with each and elects the one with the fewest errors — handy for legacy SQL scripts of mixed origin.

> The authoritative list of dialects available in your build is the in-product dropdown or `GET /api/dialects`.

### 4.4 What SQL can be analyzed

**Queries & DML**

- `SELECT` (subqueries, correlated subqueries, recursive CTEs, window functions)
- `INSERT INTO … SELECT`, `UPDATE … SET`, `DELETE`, `MERGE`
- `UNION / UNION ALL`
- Hive/Spark `LATERAL VIEW`, ClickHouse `ARRAY JOIN`

**Tables & views**

- `CREATE TABLE AS SELECT` (CTAS), `CREATE VIEW AS SELECT`, materialized views
- DDL schemas are extracted automatically for column disambiguation and `SELECT *` expansion
- `ALTER TABLE … RENAME` produces "rename" lineage

**CTEs & temp tables**

- `WITH` clauses (CTEs) render as their own cards, expanded layer by layer
- Temp tables (T-SQL `#temp`, `CREATE TEMP TABLE`) are tracked normally

**Stored procedures / functions / packages** (deep parsing)

- Oracle PL/SQL (anonymous blocks, named procedures, packages), PostgreSQL PL/pgSQL, SQL Server T-SQL (including `GO` batches), MySQL stored procedures
- Tracks **local variables**, **cursors** (`DECLARE` / `FETCH INTO`), and DML inside `IF / LOOP / FOR` control flow
- **Cross-procedure calls**: data flow through `CALL proc_b(...)` is threaded through
- **Dynamic SQL detection**: `EXECUTE IMMEDIATE`, `sp_executesql`, `PREPARE` etc. cannot be statically expanded; Esther emits a `DYNAMIC_SQL` warning and marks affected nodes with confidence 0.3

**Triggers**

- Three-layer model: ① a **binding edge** from the source table to the trigger function (⚡ badge; hover shows e.g. `BEFORE INSERT FOR EACH ROW`); ② **data flow** from `NEW.col / OLD.col` in the body to target tables; ③ **filter edges** from the body DML's WHERE clause to the target container

**Files & import/export**

- `COPY … TO / FROM 'file'` produces table ↔ file lineage; the file card shows path, format (CSV/PARQUET/JSON…), delimiter and import/export direction

### 4.5 How to read the lineage graph

#### Cards (containers)

The graph groups by "container": each table / view / CTE / procedure renders as one card listing its columns, parameters and variables, color-coded by type:

| Card type | Example | Color |
|-----------|---------|-------|
| Table | `orders` | Blue |
| View | `user_view` | Cyan |
| Materialized view | `mv_sales` | Dark cyan |
| CTE | `recent` | Green |
| Subquery | `(SELECT …) s` | Purple |
| Temp table | `#temp` | Gray-blue |
| Procedure / function | `sp_update` | Deep purple |
| File | `orders.dat` | Brown |
| Cursor / anonymous block / event | `c`, `BEGIN…END`, `CREATE EVENT` | Orange / gray / cyan |

#### Card entries

Each column entry may carry: data type, primary key (PK) / foreign key (FK) / partition key (P) / distribution key (D) badges, parameter direction (IN/OUT/INOUT/RETURN), a **confidence percentage**, and the `𝑓` expression badge for computed columns.

#### Edge types (9)

| Type | Meaning | Line style | Badge (hover content) |
|------|---------|-----------|-----------------------|
| `DIRECT` | Value passed unchanged | Gray solid | — |
| `TRANSFORM` | 1:1 computation (`price * qty`) | Purple dashed | `𝑓` (expression) |
| `AGGREGATION` | N:1 aggregation (`SUM(x) GROUP BY …`) | Orange double | `Σ` (expression + grouping) |
| `UNION` | Multi-source merge | Cyan wavy | — |
| `JOIN` | Join condition (column ↔ column) | Green dash-dot | `⋈` (`ON` condition) |
| `FILTER` | Filter condition (column → table card) | Red dotted | `⊳` (`WHERE` condition) |
| `RENAME` | Table/column rename | Violet long dash | "Renamed" label |
| `TRIGGER` | Trigger binding (table card → trigger card) | Amber dashed | `⚡` (event info) |
| `BRANCH` | Branch execution path | Yellow | — |

Open the **Legend** popover on the page to compare line styles and colors at any time.

#### Structural relations (thin lines)

Besides data flow, structural relations — foreign keys (FOREIGN_KEY), synonyms (SYNONYM_OF), partitions (PARTITION_OF) — render as translucent thin lines, clearly distinct from data-flow edges.

#### Confidence

| Value | Meaning |
|-------|---------|
| 100% | DDL-defined or explicit column list — certain |
| 80% | Inferred from the FROM clause |
| 70% | SQL inside branches (IF/CASE) |
| 30% | Involves dynamic SQL that static analysis cannot expand |
| 0% | Placeholder column name (e.g. `INSERT` without a column list) |

#### Warnings

Parse-time notices are graded by severity; clicking a warning **locates it in the SQL source**:

| Code | Meaning |
|------|---------|
| `PARSE_ERROR` | A statement failed to parse; its lineage may be missing |
| `RESOLVE_ERROR` | A statement could not produce complete lineage |
| `DYNAMIC_SQL` | Dynamic SQL detected; affected lineage marked confidence 0.3 |
| `GOTO_UNCERTAINTY` | A GOTO makes static determination impossible; conservatively marked |

### 4.6 Graph interactions

| Action | Effect |
|--------|--------|
| **Click a column / card title** | Highlights the full chain in **both directions** — all upstream origins and downstream targets light up, the rest fades |
| **Click an edge or badge** | The SQL editor selects and highlights the fragment that produced the edge |
| **Hover a badge** (`𝑓` `Σ` `⊳` `⋈` `⚡`) | Popup with the expression / condition / event details |
| **Drag cards / zoom / minimap** | Free layout adjustment; auto-layout is left-to-right layered |
| **Export PNG** | Save the current graph as an image for documents |
| **JSON tab** | Structured result identical to the API |

### 4.7 Upstream tracing & downstream impact analysis

This is the persistent form of the lineage graph, answering "if I change this column, what breaks?":

1. **Build persistent lineage**: configure a database connection on the Data Sources page and run a **metadata ingestion pipeline**. After ingestion, Esther automatically parses the views / materialized views (plus procedures and triggers) in the database, saves the lineage into the embedded graph database (Kuzu), and creates a **lineage snapshot** (node/edge counts, version, affected objects).
2. **Browse**: open any data asset's detail page and switch to the **Lineage Graph** tab:
   - **Full lineage** — upstream + downstream
   - **Upstream tracing** — where the data comes from, layer by layer back to origins
   - **History / snapshots** — compare lineage across ingestion versions
3. **Multi-hop traversal**: tracing defaults to unlimited depth (system cap: 30 hops); control it with the API's `depth` parameter.

> Relationship to the analysis page (Chapter 3): that page is an ad-hoc sandbox that does not write to the graph database; only pipeline-generated lineage is persisted and participates in cross-object tracing.

### 4.8 Metadata assistance: making lineage more accurate

Three ways to feed schema information to the parser:

| Method | How | What it solves |
|--------|-----|----------------|
| **Inline schemas** | Pass `table_schemas` when calling the API | Expands `SELECT *` by real columns; disambiguates same-named columns |
| **Paste DDL** | Include `CREATE TABLE` statements in the SQL | Same, with no extra parameter |
| **Metadata ingestion** | Configure a data source and run a pipeline | Whole-database parsing with view-definition traversal (view → view → table) |

Example `table_schemas` usage: [5.1](#51-analysis-endpoint).

---

## 5. REST API Reference

Interactive API documentation (Swagger UI): `http://<server-ip>:8000/docs`

### 5.1 Analysis endpoint

Endpoint: `POST /api/analyze`

**Request body**

```json
{
  "sql": "INSERT INTO report (id, total) SELECT order_id, price * qty FROM orders",
  "dialect": "mysql",
  "project": "default",
  "table_schemas": {
    "orders": {
      "columns": {
        "order_id": {"data_type": "INT", "is_primary_key": true},
        "price":    {"data_type": "DECIMAL(10,2)"},
        "qty":      {"data_type": "INT"}
      }
    }
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `sql` | ✅ | SQL text; may contain multiple statements, DDL, procedures |
| `dialect` | — | Dialect ID, defaults to `auto` (auto-detect) |
| `project` | — | Project name, defaults to `default` |
| `table_schemas` | — | Schema dictionary keyed by table name/FQN, for `SELECT *` expansion and disambiguation |

**Response (excerpt)**

```json
{
  "success": true,
  "data": {
    "analysis_id": "a1b2c3d4…",
    "dialect": "mysql",
    "nodes": [
      { "id": "…", "type": "COLUMN", "name": "order_id",
        "fqn": "orders.order_id", "parent_id": "…", "confidence": 0.8 },
      { "id": "…", "type": "COLUMN", "name": "total",
        "fqn": "report.total", "metadata": { "transform_expr": "price * qty" } }
    ],
    "edges": [
      { "id": "…", "type": "TRANSFORM",
        "source_id": "…orders.price", "target_id": "…report.total",
        "transform_expr": "price * qty",
        "sql_start": 42, "sql_end": 54 }
    ],
    "relations": [],
    "warnings": []
  }
}
```

Node `type` is the node type (COLUMN / PARAMETER / VARIABLE / LITERAL …); edge `type` values are listed in [4.5](#45-how-to-read-the-lineage-graph).

### 5.2 Lineage query endpoints (login token required)

| Endpoint | Purpose |
|----------|---------|
| `GET /api/v1/lineage/{fqn}` | Query upstream/downstream lineage of an asset by FQN |
| `GET /api/v1/lineage/manifest` | Parameterized query by data source + level (database/schema/table/column) |
| `GET /api/v1/lineage/snapshots` | Lineage snapshot list |
| `GET /api/v1/lineage/snapshots/{id}` | Snapshot detail |

**Common query parameters**

| Parameter | Values | Description |
|-----------|--------|-------------|
| `direction` | `upstream` / `downstream` / `both` | Traversal direction, default both |
| `depth` | `-1`–`30` | Traversal depth; `-1` = unlimited (default), cap 30 |
| `asset_type` | `database` / `schema` / `table` / `view` / `column` | Asset level |
| `project` | project name | Restrict to a project |

### 5.3 Other endpoints

| Endpoint | Purpose |
|----------|---------|
| `GET /api/dialects` | All available dialects and aliases (common ones first) |
| `POST /api/v1/auth/login` | Login to obtain a JWT token |
| `/api/v1/datasources` | Data source CRUD, connection testing, metadata sync |

> ⚠️ `POST /api/analyze` has no authentication — use it on internal networks only; `/api/v1/*` requires a login token.

---

## 6. CLI Reference

The CLI lives in the program directory (`esther-cli.exe` on Windows) and is suited to scripted batch analysis.

### 6.1 Analyzing SQL

```bash
# Analyze a SQL string, table output
esther-cli.exe analyze --dialect mysql --sql "SELECT a.id FROM orders a"

# Analyze a SQL file, JSON output (redirect to save)
esther-cli.exe analyze --dialect auto --file query.sql --format json > result.json

# Specify project and graph DB path
esther-cli.exe analyze -d postgres -s "SELECT * FROM t" --project etl --db-path esther.db
```

| Parameter | Description |
|-----------|-------------|
| `--dialect / -d` | Dialect ID; supports `auto` and all aliases |
| `--sql / -s` | SQL text |
| `--file / -f` | Path to a SQL file |
| `--format` | `json` or `table` (default) |
| `--project / -p` | Project name, default `default` |
| `--db-path` | Graph database file path |
| `--config / -c` | Config file path; defaults to `esther.toml` next to the executable |

### 6.2 Other commands

```bash
esther-cli.exe dialects list        # List all dialects and aliases
esther-cli.exe project create myprj # Create a project
esther-cli.exe project list         # List projects
esther-cli.exe project export myprj # Export all analyses in a project as JSON
esther-cli.exe license status       # Show license status
esther-cli.exe license fingerprint  # Print the machine fingerprint
```

### 6.3 Exit codes

| Code | Meaning |
|------|---------|
| `0` | Success, no warnings |
| `1` | Success with parse warnings |
| `2` | Error (unknown dialect / missing file / bad format) |
| `3` | License blocked (trial expired or unlicensed) |

---

## 7. Configuration

The configuration file is `esther.toml` in the program directory.

### 7.1 Storage backend

```toml
[storage]
# Default: kuzu — embedded graph database, zero deployment. Optional: neo4j.
backend = "kuzu"

[storage.kuzu]
db_path = "./esther.db"

# Switch to Neo4j (optional)
# [storage.neo4j]
# uri      = "bolt://localhost:7688"
# user     = "neo4j"
# password = "secret"
# database = "esther"
```

### 7.2 Server port

Change via the `ESTHER_PORT` environment variable (default `8000`); restart to apply.

### 7.3 Lineage verifier agent (optional, advanced)

`esther.toml` ships with a reserved **dual-model lineage verification** configuration: two independent LLMs act as predictor and arbiter to spot-check lineage results. It requires the corresponding API keys in environment variables (`DEEPSEEK_API_KEY`, `DASHSCOPE_API_KEY`); regular usage does not need it.

---

## 8. FAQ & Known Limitations

**Q: Auto-detection picked the wrong dialect.**
A: Mixed-dialect scripts can be misdetected — specify the dialect manually. Confirm available dialects via `GET /api/dialects`.

**Q: Can dynamic SQL (`EXECUTE IMMEDIATE` / `sp_executesql`) be analyzed?**
A: SQL assembled at runtime cannot be expanded statically. Esther detects it, emits a `DYNAMIC_SQL` warning and marks affected nodes at 30% confidence — nothing is silently dropped.

**Q: My analysis-page results disappeared after refresh.**
A: The analysis page is an ad-hoc sandbox and does not persist. Use ingestion pipelines for long-lived, cross-object lineage ([4.7](#47-upstream-tracing--downstream-impact-analysis)).

**Known limitations**

| Limitation | Details |
|------------|---------|
| Traversal depth cap | At most 30 hops (covers virtually all real chains) |
| Auto-layout layers | At most 12 layers per graph; for very large graphs, browse per asset |
| `POST /api/analyze` unauthenticated | Keep it on internal networks |
| GOTO static uncertainty | DML skipped by GOTO is flagged with `GOTO_UNCERTAINTY` and conservatively marked |
| Antivirus false positives | Nuitka builds may trigger some antiviruses — allowlist the program |

---

## 9. Getting Help

- **Bug reports / feature requests**: open an issue on [GitHub Issues](https://github.com/estherdata2026/esther-official-resources/issues) (please use a template) or [Gitee Issues](https://gitee.com/esther2026/esther-official-resources/issues). To speed up diagnosis, please include:
  1. The **original SQL** (complete statements; anonymize values but keep the structure)
  2. The dialect ID
  3. Expected vs. actual lineage (screenshot or JSON)
  4. Your version (from `esther-cli.exe license status` or the startup log)
- **Licensing & business**: email [estherdata@163.com](mailto:estherdata@163.com), or leave a message via an issue.

---

*Esther — The Wise Guardian of Your Data | 您的数据智慧管家*
