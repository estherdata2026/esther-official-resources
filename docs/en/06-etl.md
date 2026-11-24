# 6. ETL Tool Lineage Analysis


Beyond SQL, Esther parses job files from mainstream ETL tools directly and produces column-level lineage graphs **isomorphic** to SQL lineage — the same graph interactions, the same query API, the same graph database. What happens "inside the ETL tool" is no longer a black box.

### 6.1 Supported ETL tools

| ETL tool | Job file | Notes |
|----------|----------|-------|
| Kettle (Pentaho) | `.ktr` (transformation) / `.kjb` (job) | Step SQL is parsed in its dialect and merged into the lineage |
| SSIS | `.dtsx` package | |
| DataX | job `.json` config | Custom UDFs can be registered as transform / passthrough |
| Informatica PowerCenter | exported XML (or extension-less export) | Extension-less files require selecting the tool manually |
| dbt | `manifest.json` (the `target/` build artifact) | Optionally combined with `catalog.json` |

- File size limit: **20MB**.
- The tool type is **auto-detected** from file content by default; manual selection is also available.
- Fully regression-tested against 355 real jobs from the official Kettle sample repository.

### 6.2 How to use (Web UI)

1. Open the **Data Lineage** page in **ETL mode** (the menu offers a direct entry, or visit `/lineage?mode=etl`).
2. Pick the ETL tool (auto-detect by default), then **upload a file** or **paste the content**.
3. Click **Analyze ETL**.
4. Review the results:
   - **Step strip**: the list of steps in the job (name / type / category / dialect / connection / target table); click a step to highlight its lineage edges in the graph;
   - **Graph / Table / JSON**: the same three views as the SQL analysis page;
   - **Dialect report**: which dialect each connection resolved to and how (explicit / connection-type / URL-inferred / default fallback).
5. Connections whose dialect cannot be detected: the page lists them; pick a dialect for each and **re-analyze with dialects** to include their step SQL in the lineage.

> Advanced options: pre-assign dialects to undetected connections and set a fallback dialect; for dbt, attach `catalog.json` for richer column types and comments.

### 6.3 REST API

| Endpoint | Description |
|----------|-------------|
| `POST /api/v1/etl/analyze` | JSON body: `content` (file content) + `path` (file name) + `tool` (default `auto`), optional `default_dialect` / `connection_dialects` / `transformer_udfs` / `catalog_json` |
| `POST /api/v1/etl/analyze/upload` | multipart upload; parameters passed as form fields; for dbt attach the `catalog` file |

The response is isomorphic to the SQL analysis response (`nodes` / `edges` / `warnings` / `metadata_tree`) plus `tool`, `job` (job & step outline) and `dialect_report`. ETL lineage is persisted into the graph database as well (analysis IDs prefixed with `etl:`) and can be traced via the [lineage query endpoints](#82-lineage-query-endpoints-login-token-required).

> ETL lineage analysis is gated by the license feature `etl`; without it the API returns 403 with application guidance instead of failing silently.

### 6.4 Known boundaries

- Step SQL follows the parsing capability of its SQL dialect (see [4.3](#43-supported-sql-dialects));
- Steps whose connection has no resolved dialect are excluded from lineage until a dialect is supplied and the job is re-analyzed;
- Disabled hops produce no lineage; empty flows produce explicit warnings.

---
