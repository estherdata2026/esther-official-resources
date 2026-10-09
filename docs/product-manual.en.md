# Esther Product Manual

**Esther · SQL Data Lineage Analysis Platform | The Wise Guardian of Your Data**

| | |
|------|------|
| Applies to | 0.12.0-preview (Windows preview build) |
| Last updated | 2026-10-09 |
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
   - [4.6 Graph interactions & lineage queries](#46-graph-interactions--lineage-queries)
   - [4.7 Upstream tracing & downstream impact analysis](#47-upstream-tracing--downstream-impact-analysis)
   - [4.8 Metadata assistance: making lineage more accurate](#48-metadata-assistance-making-lineage-more-accurate)
5. [Metadata Extraction Pipelines (Persistent Lineage)](#5-metadata-extraction-pipelines-persistent-lineage)
6. [REST API Reference](#6-rest-api-reference)
7. [CLI Reference](#7-cli-reference)
8. [Configuration](#8-configuration)
9. [FAQ & Known Limitations](#9-faq--known-limitations)
10. [Getting Help](#10-getting-help)

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
- 27 SQL dialects (including GaussDB) covering mainstream databases and Chinese domestic databases, with automatic detection
- Deep procedure parsing: PL/SQL, PL/pgSQL, T-SQL, MySQL procedures — variables, cursors, control flow, triggers
- Interactive lineage queries: direction / depth / type filters / endpoint tracing, one-click table-level vs. column-level views, full-path highlighting on selection
- Dual lineage views: interactive graph + spreadsheet-style data table (filter / sort / paginate, CSV export); graph export as PNG, results as JSON
- Metadata extraction pipelines: dual sources (data source connections + SQL script packages), diff-based merging that adds, updates and removes assets automatically, run logs and lineage snapshots
- Upstream tracing / downstream impact analysis backed by an embedded graph database

---

## 2. Installation & Licensing

### 2.1 Deploy in three steps (Windows preview build)

1. Download the zip package from [GitHub Releases](https://github.com/estherdata2026/esther-official-resources/releases) or [Gitee releases](https://gitee.com/esther2026/esther-official-resources/releases) and extract it anywhere on the target server (portable, no installer).
2. Double-click `start_esther.bat` to start the server.
3. Open `http://<server-ip>:8000` in your browser and sign in with the administrator account.

**Changing the port**: set the `ESTHER_PORT` environment variable and restart.

### 2.2 Licensing

- **Trial mode**: the first run starts a trial period (90 days by default); all features are available during the trial, and the server stops starting once it expires. Remaining days are shown anytime by `esther-cli.exe license status`.
- **Activating a license**:
  1. Run `esther-cli.exe license fingerprint` to get the machine fingerprint;
  2. Email the fingerprint to [estherdata@163.com](mailto:estherdata@163.com) to receive the license file `esther.lic`;
  3. Place `esther.lic` in the program directory and restart — the server enters registered mode.
- **Checking status**: `esther-cli.exe license status`.

### 2.3 Runtime data & accounts

- Runtime data is created automatically under `data/` (SQLite + Kuzu graph database) — no external database required.
- On first startup an administrator account **admin** (initial password **admin123**) is created automatically — keep it safe after signing in.
- Licenses are bound to the machine fingerprint; migrating servers requires a new license.

### 2.4 Security notes

- The program is a Nuitka native build; some antivirus products may false-positive on it — add it to the allowlist. Dependencies are statically linked; no VC++ runtime installation is required.
- `POST /api/analyze` currently has no authentication — deploy on an **internal network**. The `/api/v1/*` endpoints require a login token.

---

## 3. Quick Start: Your First Lineage Analysis

After opening the system in your browser:

1. Sign in on the login page with the administrator account (see [2.3](#23-runtime-data--accounts) for the default).
2. Click **Data Lineage** (**数据血缘**) in the navigation bar. On first load the page **automatically runs a demo SQL** and renders the lineage graph — you see results immediately.
3. Paste your own SQL into the editor (multiple statements, stored procedures and CREATE statements are all fine).
4. Pick the dialect from the dropdown; leave **Auto-detect** if unsure.
5. Click **Analyze** (shortcut: `Ctrl + Enter`).
6. Inspect the results:
   - Stats bar: **node / edge / warning counts** and the detected dialect
   - **Lineage graph** tab: the interactive graph; the toolbar supports direction / depth / type filters / endpoint tracing (see [4.6](#46-graph-interactions--lineage-queries))
   - **Table** tab: a spreadsheet-style lineage data table with filtering, sorting, pagination and CSV export
   - **JSON** tab: the raw result, identical to the API response
   - Warnings panel: click a warning to locate it in the SQL

> Note: the analysis page is an ad-hoc sandbox — results are not persisted. For persistent lineage that supports cross-object tracing, use metadata extraction pipelines (see Chapter 5).

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
| **Output** | Interactive graph + data table + JSON | JSON (same as the UI's JSON tab) | JSON or terminal tables |
| **See** | Chapter 3 | Chapter 6 | Chapter 7 |

All three share the same parsing engine and produce identical result structures.

### 4.3 Supported SQL dialects

Esther supports **27** SQL dialects. Common aliases (e.g. `pg`, `mysql`) are accepted as dialect identifiers.

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

### 4.6 Graph interactions & lineage queries

**Interaction model**: clicking a node / card only **selects and highlights** it — queries are never triggered automatically. Lineage queries start explicitly from the toolbar buttons or the right-click menu.

#### Selection & full-path highlighting

- **Click a column / card title**: highlights **every path that passes through it** in the current graph — all upstream origins and downstream targets light up, the rest fades.
- **Click an edge or badge**: highlights that edge plus its upstream/downstream chains, and the SQL editor selects the fragment that produced the edge.
- **Hover a badge** (`𝑓` `Σ` `⊳` `⋈` `⚡`): popup with the expression / condition / event details.

#### Lineage query toolbar

| Control | Effect |
|---------|--------|
| **Direction: Upstream / Downstream / Both** | Traverses lineage server-side from the selected node in the chosen direction (also available in the right-click menu) |
| **Depth** | Numeric input; by default it runs to the end and shows the actually reachable maximum depth — type a number to cut the traversal off |
| **Simplified lineage: Table / Column** | One-click graph density: **Column** (default) shows column-level lineage, **Table** keeps only table / view level relations |
| **Custom (type filter)** | Choose which node types (table / view / procedure / CTE / …) and edge types are shown; optionally keep isolated nodes; one-click reset. Containers at both ends of a chain (origins / outputs) are never omitted, keeping chains readable |
| **⇤ Furthest origins / ⇥ Final destinations** | Endpoint tracing: keep only the start node plus the ultimate sources / sinks, omitting everything passed through |
| **Reset** | Back to the initial view (restores column-level simplification, clears selection and queries) |
| **Fullscreen** | View large graphs full-screen |

**Traversal semantics**: direction queries traverse only **data-flow edges** (direct / transform / aggregation / union); conditional lines such as JOIN and FILTER are display-only and never followed — results contain real data flow only. "Both" is the union of two independent one-direction traversals.

#### Table tab

The "Table" tab shows the complete result of the current analysis in a spreadsheet-style grid:

- **Lineage relations** dataset: one row per edge — source object / source column / target object / target column / edge type / transform expression / confidence;
- **Nodes** dataset: name / type / FQN / parent container / data type / confidence, etc.;
- Sorting, per-column filter rows, pagination (100 rows per page by default) and cell copy;
- **Export CSV** — exports rows as currently filtered and sorted.

> The table always shows the complete analysis result; it does not follow the query lens (direction / depth / type filter) applied to the graph.

### 4.7 Upstream tracing & downstream impact analysis

This is the persistent form of the lineage graph, answering "if I change this column, what breaks?":

1. **Build persistent lineage**: **metadata extraction pipelines** (Chapter 5) save lineage from data-source metadata and SQL scripts into the embedded graph database (Kuzu), creating a **lineage snapshot** after every effective change.
2. **Browse & query**: open any data asset's detail page and switch to the **Lineage** tab. The query toolbar is identical to the analysis page (direction / depth / endpoint tracing / type filters), traversing server-side from that asset; database / schema assets have no single start point and show the whole graph with a "lineage objects (N)" stat.
3. **Snapshot comparison**: every effective change creates a new lineage version, which you can diff against previous ingestion versions.
4. **Multi-hop traversal**: tracing runs to full depth by default (no hard cap); control it with the API's `depth` parameter (see [6.2](#62-lineage-query-endpoints-login-token-required)).

> Relationship to the analysis page (Chapter 3): that page is an ad-hoc sandbox that does not write to the graph database; only pipeline-generated lineage is persisted and participates in cross-object tracing.

### 4.8 Metadata assistance: making lineage more accurate

Three ways to feed schema information to the parser:

| Method | How | What it solves |
|--------|-----|----------------|
| **Inline schemas** | Pass `table_schemas` when calling the API | Expands `SELECT *` by real columns; disambiguates same-named columns |
| **Paste DDL** | Include `CREATE TABLE` statements in the SQL | Same, with no extra parameter |
| **Metadata extraction** | Configure a data source or SQL package and run a pipeline | Whole-database parsing with view-definition traversal (view → view → table) |

Example `table_schemas` usage: [6.1](#61-analysis-endpoint).

---

## 5. Metadata Extraction Pipelines (Persistent Lineage)

Extraction pipelines bring external metadata and SQL scripts into the platform: **extracting metadata** builds the data asset catalog, **parsing lineage** writes into the graph database, with scheduled runs and diff-based merging.

### 5.1 Two sources (combinable; at least one required)

| Source | Content | Notes |
|--------|---------|-------|
| **Data source** | A database connection configured on the Data Sources page | Extracts tables / columns / views / materialized views / procedures / functions / triggers / sequences / synonyms / packages / indexes / constraints / partitions etc. into the asset catalog |
| **SQL package (zip)** | A zip of `.sql` / `.ddl` / `.txt` scripts | Parses DDL and DML: `CREATE TABLE` becomes data assets, and every statement is mined for column-level lineage. Limits: ≤ 20MB per package, ≤ 500 files, ≤ 100MB uncompressed; a single file's failure never aborts the batch; multiple file encodings are tried automatically |

**When both are selected**: the data source's latest metadata is extracted first, then the script lineage is parsed against it (script tables resolve against real structures, making `SELECT *` expansion and disambiguation more accurate).

### 5.2 Creating & editing a pipeline

On the "Extraction Pipelines" page, click create and fill in:

- **Name** (required);
- **Data source**: pick a configured connection;
- **SQL package**: upload a zip;
- **Database dialect**: **required for script packages — "Auto-detect" is not allowed**. Auto-detection over a whole package is prone to misjudgment, which would cause missing parses and silently absent assets, so the database type must be stated explicitly;
- **Schedule**: manual / hourly / daily at midnight / daily at 2am / Sundays at 3am (cron presets);
- **Description**.

Pipelines can be **edited** at any time: swap the data source, re-upload the package, or change the dialect / schedule / name / description.

### 5.3 Running & scheduling

- **Run now**: click "Run now" on the pipeline card or detail page;
- **Scheduled runs**: automatic runs on the preset cron cadence (the scheduler scans for due pipelines every minute);
- Pipelines targeting the same data source **run exclusively**: while one is running, others on the same source are asked to retry later.

### 5.4 Diff-based merging: no change, no new version

Each run compares the freshly extracted truth against the asset catalog via content fingerprints (object name → content digest):

- **No difference**: the run is marked "**No changes**" and produces no new asset version or lineage snapshot (lineage is still rebuilt from scratch, staying consistent with the latest metadata);
- **Differences**: assets that disappeared from the source are **deleted automatically** (dropping a source table no longer leaves residue) → a metadata snapshot is created → lineage is re-parsed → a lineage snapshot is created;
- Run history shows the change summary as **`+N ~N -M`** (added / updated / removed).

### 5.5 Run history & logs

The run history on the pipeline detail page records, per run:

- **Asset changes**: `+N ~N -M` or "No changes";
- **Lineage output**: `N nodes · M edges` produced by the run;
- **Error badge `⚠ N`**: the number of parse failures in the run — hover for the error summary (up to 20 entries); a successful run may still have some files that failed to parse;
- **View log**: a dialog with the full run log (including per-file parse errors), **downloadable as a `.log` file** for troubleshooting feedback.

### 5.6 Adopting orphan tables

Tables that appear in lineage but not in the asset catalog (possible typos, dynamic SQL, or simply not yet ingested) are flagged as **`◇ N`** in the run history. Expand to adopt them one by one, or **adopt all** — confirmed adoptions become regular assets.

### 5.7 Views & sequences

- **Views become assets**: `CREATE VIEW` / `CREATE MATERIALIZED VIEW` (including statements inside script packages) registers a **view asset** with its definition text — lineage parsing traverses through definitions layer by layer (view → view → table), and `SELECT *` expands correctly against view output columns;
- **Sequences (SEQUENCE)**: a sequence is a counter, not a data-flow object — it is recorded as its own node type with attributes (start / increment / min / max), no fake table cards, and no column-level lineage.

### 5.8 Asset catalog & asset tree

- Assets are organized as **database → schema → object**; objects in script packages missing database/schema qualifiers are filed under the `default` namespace so the asset tree never breaks;
- The **Structure** list on an asset's detail page shows its children (table → columns, schema → tables / views) for drill-down;
- In the analysis page's metadata tree, **tables / views (incl. materialized views) / columns** support right-click lineage queries (downstream / upstream / both / endpoint tracing) — the same actions as the graph's right-click menu.

---

## 6. REST API Reference

Interactive API documentation (Swagger UI): `http://<server-ip>:8000/docs`

### 6.1 Analysis endpoint

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

### 6.2 Lineage query endpoints (login token required)

| Endpoint | Purpose |
|----------|---------|
| `POST /api/v1/lineage/query` | Lineage view query: direction / depth / type filters / endpoint tracing, traversed server-side |
| `GET /api/v1/lineage/snapshots` | Lineage snapshot list |
| `GET /api/v1/lineage/snapshots/{id}` | Snapshot detail |
| `GET /api/v1/lineage/batches` | Batch parse job list |
| `GET /api/v1/lineage/batches/{id}` | Batch detail |
| `POST /api/v1/lineage/upload-batch` | REST batch parsing of SQL packages (valid license required) |

**`POST /api/v1/lineage/query` request body**

```json
{
  "analysis_id": "a1b2c3d4…",
  "view": {
    "start": { "type": "column", "id": "…orders.price" },
    "direction": "downstream",
    "depth": -1,
    "show_node_types": ["TABLE", "VIEW", "COLUMN"],
    "endpoints": null
  }
}
```

| Field | Values | Description |
|-------|--------|-------------|
| `analysis_id` / `datasource_id` | exactly one | Analysis-scoped query (analysis page / sandbox results) or datasource-scoped query (optionally with `schema_name`) |
| `view.start` | `{type, id}` or omitted | Start point (container or column); **omit = whole-graph view** (combine with type filters to slice the entire graph) |
| `view.direction` | `upstream` / `downstream` / `both` | Traversal direction, default `both` |
| `view.depth` | `-1` or positive integer | Traversal depth; `-1` = unlimited (default); depth has no hard cap |
| `view.show_node_types` / `view.show_edge_types` | type whitelist or omitted | Show only the listed node / edge types; omitted = no filtering |
| `view.endpoints` | `upstream` / `downstream` / omitted | Endpoint tracing: return only the start node plus furthest origins / final destinations |
| `view.show_isolated` | boolean, default `false` | In a whole-graph view, keep nodes without any lineage relations |

**Response (excerpt)**: `nodes` / `edges` (same structure as [6.1](#61-analysis-endpoint)) plus `stats` (`result_nodes` / `result_edges` / `result_containers` …), `truncated` (whether the result was cut off) and `max_depth` (the actually reachable maximum depth from the start point).

> Migration note: the old `GET /api/v1/lineage/{fqn}` and `GET /api/v1/lineage/manifest` endpoints have been replaced by `POST /api/v1/lineage/query`; existing integrations should move to the new endpoint.

### 6.3 Extraction pipeline endpoints (login token required)

| Endpoint | Purpose |
|----------|---------|
| `POST /api/v1/pipelines/upload-zip` | Upload a SQL package, returns `zipFileId` |
| `POST /api/v1/pipelines` | Create a pipeline (data source and/or SQL package) |
| `GET /api/v1/pipelines`, `GET /api/v1/pipelines/{id}` | Pipeline list / detail |
| `PUT /api/v1/pipelines/{id}` | Edit a pipeline (data source / package / dialect / schedule …) |
| `POST /api/v1/pipelines/{id}/run` | Run now (same-source exclusivity applies) |
| `GET /api/v1/pipelines/{id}/runs` | Run history (change summary, lineage output, error counts) |
| `GET /api/v1/pipelines/{id}/runs/{run_id}/log` | View a run log (`…/log/download` to download) |
| `POST /api/v1/pipelines/{id}/runs/{run_id}/orphans/adopt` | Adopt orphan tables into the asset catalog |

### 6.4 Other endpoints

| Endpoint | Purpose |
|----------|---------|
| `GET /api/dialects` | All available dialects and aliases (common ones first) |
| `POST /api/v1/auth/login` | Login to obtain a JWT token |
| `/api/v1/datasources` | Data source CRUD, connection testing, metadata sync (sync merges by diff, adding/updating/removing assets automatically) |
| `GET /api/v1/projects`, `POST /api/v1/projects` | Project list / creation |
| `GET /api/v1/auth/assets/{fqn}/children` | Child structure list of an asset |

> ⚠️ `POST /api/analyze` has no authentication — use it on internal networks only; `/api/v1/*` requires a login token; `POST /api/v1/lineage/upload-batch` (batch lineage generation) requires a valid license.

---

## 7. CLI Reference

The CLI lives in the program directory (`esther-cli.exe` on Windows) and is suited to scripted batch analysis.

### 7.1 Analyzing SQL

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

### 7.2 Other commands

```bash
esther-cli.exe dialects list        # List all dialects and aliases
esther-cli.exe project create myprj # Create a project
esther-cli.exe project list         # List projects
esther-cli.exe project export myprj # Export all analyses in a project as JSON
esther-cli.exe license status       # Show license status
esther-cli.exe license fingerprint  # Print the machine fingerprint
```

### 7.3 Exit codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Success with parse warnings; also returned when `license status` detects a blocked license |
| `2` | Error (unknown dialect / missing file / bad format) |
| `3` | License blocked (trial expired or unlicensed) |

---

## 8. Configuration

The configuration file is `esther.toml` in the program directory.

### 8.1 Storage backend

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

### 8.2 Server port

Change via the `ESTHER_PORT` environment variable (default `8000`); restart to apply.

### 8.3 Lineage verifier agent (optional, advanced)

`esther.toml` ships with a reserved **dual-model lineage verification** configuration: two independent LLMs act as predictor and arbiter to spot-check lineage results. It requires the corresponding API keys in environment variables (`DEEPSEEK_API_KEY`, `DASHSCOPE_API_KEY`); regular usage does not need it.

---

## 9. FAQ & Known Limitations

**Q: Auto-detection picked the wrong dialect.**
A: Mixed-dialect scripts can be misdetected — specify the dialect manually. Confirm available dialects via `GET /api/dialects`.

**Q: Can dynamic SQL (`EXECUTE IMMEDIATE` / `sp_executesql`) be analyzed?**
A: SQL assembled at runtime cannot be expanded statically. Esther detects it, emits a `DYNAMIC_SQL` warning and marks affected nodes at 30% confidence — nothing is silently dropped.

**Q: My analysis-page results disappeared after refresh.**
A: The analysis page is an ad-hoc sandbox and does not persist. Use extraction pipelines for long-lived, cross-object lineage (Chapter 5).

**Q: Why must a SQL package in an extraction pipeline declare a dialect?**
A: Auto-detection over a whole package is prone to misjudgment, which would silently drop parses and assets. Script packages therefore require an explicit database type, double-checked at run time.

**Known limitations**

| Limitation | Details |
|------------|---------|
| Traversal depth | No hard cap (runs to full depth by default); for very large graphs, narrow the view with type filters / endpoint tracing |
| Auto-layout layers | At most 12 layers per graph; for very large graphs, browse per asset |
| `POST /api/analyze` unauthenticated | Keep it on internal networks |
| GOTO static uncertainty | DML skipped by GOTO is flagged with `GOTO_UNCERTAINTY` and conservatively marked |
| Antivirus false positives | Nuitka builds may trigger some antiviruses — allowlist the program |

---

## 10. Getting Help

- **Bug reports / feature requests**: open an issue on [GitHub Issues](https://github.com/estherdata2026/esther-official-resources/issues) (please use a template) or [Gitee Issues](https://gitee.com/esther2026/esther-official-resources/issues). To speed up diagnosis, please include:
  1. The **original SQL** (complete statements; anonymize values but keep the structure)
  2. The dialect ID
  3. Expected vs. actual lineage (screenshot or JSON)
  4. Your version (from `esther-cli.exe license status`, the startup log, or `GET /api/health`)
- **Licensing & business**: email [estherdata@163.com](mailto:estherdata@163.com), or leave a message via an issue.

---

*Esther — The Wise Guardian of Your Data | 您的数据智慧管家*
