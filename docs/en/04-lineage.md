# 4. Data Lineage Analysis (Core Feature)


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
| **See** | Chapter 3 | Chapter 8 | Chapter 9 |

All three share the same parsing engine and produce identical result structures.

### 4.3 Supported SQL dialects

Esther supports **30** SQL dialects. Common aliases (e.g. `pg`, `mysql`) are accepted as dialect identifiers.

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
| Snowflake | `snowflake` | `snowflakedb` |
| H2 | `h2` | `h2db` |
| MonetDB | `monetdb` | `monet` |

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
4. **Multi-hop traversal**: tracing runs to full depth by default (no hard cap); control it with the API's `depth` parameter (see [8.2](#82-lineage-query-endpoints-login-token-required)).

> Relationship to the analysis page (Chapter 3): that page is an ad-hoc sandbox that does not write to the graph database; only pipeline-generated lineage is persisted and participates in cross-object tracing.

### 4.8 Metadata assistance: making lineage more accurate

Three ways to feed schema information to the parser:

| Method | How | What it solves |
|--------|-----|----------------|
| **Inline schemas** | Pass `table_schemas` when calling the API | Expands `SELECT *` by real columns; disambiguates same-named columns |
| **Paste DDL** | Include `CREATE TABLE` statements in the SQL | Same, with no extra parameter |
| **Metadata extraction** | Configure a data source or SQL package and run a pipeline | Whole-database parsing with view-definition traversal (view → view → table) |

Example `table_schemas` usage: [8.1](#81-analysis-endpoint).

---
