# 5. Metadata Extraction Pipelines (Persistent Lineage)


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
