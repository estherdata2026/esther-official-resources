# 1. Product Overview


**Esther** is an enterprise **data lineage & data intelligence platform**. Beyond parsing SQL text, it reads job files from ETL tools such as Kettle, SSIS, DataX, Informatica and dbt, and extracts **column-level** data flow relationships — where a report field's data comes from, what computations and transformation steps happen along the way, and where it flows to — presented as an interactive lineage graph. On top of that, Esther ships a metadata asset catalog, a data quality engine, an AI assistant and an MCP server, turning lineage from "a picture" into live data you can query, chat with, and plug into AI workflows.

### 1.1 What problems it solves

| Scenario | Without lineage | With Esther |
|----------|-----------------|-------------|
| **Change impact analysis** | Changing a base column — no idea which downstream reports break | Select the column on the graph and see the entire downstream chain |
| **Data governance** | Relationships between tables/views/procedures buried in thousands of lines of SQL | Automatic parsing builds the full lineage graph |
| **Compliance / audit** | "Where does this external field come from?" answered by manual code reading | Upstream tracing walks back to data origins |
| **Migration / refactoring assessment** | Manual effort estimates that miss things | Quantify affected objects from the graph |
| **ETL job inventory / migration** | Hundreds of Kettle / SSIS jobs hide their logic inside the tool — nobody can see the whole picture | Upload the job files; transformation steps and column-level lineage on one graph |
| **"Ask" the data / AI integration** | Clicking through menus to find every answer | Ask the AI assistant in natural language, or query lineage from Claude Code / Cursor via MCP |

### 1.2 Usage modes

- **Web UI** — paste SQL in the browser, get an interactive graph instantly
- **REST API** — full OpenAPI/Swagger documentation, easy to integrate
- **Command line CLI** — scripted batch analysis, JSON or table output
- **AI assistant & MCP** — built-in AI Q&A and lineage diagnosis; an MCP endpoint for AI clients such as Claude Code and Cursor (see Chapter 7)

### 1.3 Capability highlights

- Column-level lineage with 9 edge types (direct / transform / aggregation / filter / join / …)
- 30 SQL dialects (including GaussDB) covering mainstream databases and Chinese domestic databases, with automatic detection
- Deep procedure parsing: PL/SQL, PL/pgSQL, T-SQL, MySQL procedures — variables, cursors, control flow, triggers
- ETL tool lineage: Kettle, SSIS, DataX, Informatica PowerCenter and dbt job files parsed directly into column-level lineage
- AI assistant: intelligent Q&A and lineage diagnosis, with pluggable model endpoints (OpenAI-compatible APIs, DeepSeek, Qwen, …)
- Built-in MCP server: AI clients such as Claude Desktop and Cursor can query lineage directly via MCP, with access token management
- Data quality engine: quality rules bound to assets, scheduled runs, asynchronous execution and Webhook alerts, with row-level sampling and reference dataset checks
- Interactive lineage queries: direction / depth / type filters / endpoint tracing, one-click table-level vs. column-level views, full-path highlighting on selection
- Dual lineage views: interactive graph + spreadsheet-style data table (filter / sort / paginate, CSV export); graph export as PNG, results as JSON
- Metadata extraction pipelines: dual sources (data source connections + SQL script packages), diff-based merging that adds, updates and removes assets automatically, run logs and lineage snapshots
- Upstream tracing / downstream impact analysis backed by an embedded graph database

---
