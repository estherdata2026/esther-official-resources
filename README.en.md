# Esther · SQL Data Lineage Analysis Platform

**English** | [简体中文](README.md)

**The Wise Guardian of Your Data**

**Esther** is an enterprise SQL data lineage analysis platform. It extracts **column-level** data flow relationships directly from SQL text, supports 30 SQL dialects (including Chinese domestic databases such as Dameng, Kingbase, OceanBase and GaussDB), and provides an interactive lineage graph with direction/depth/type-filter/endpoint lineage queries, upstream tracing and downstream impact analysis. It can also parse job files from ETL tools such as **Kettle, SSIS, DataX, Informatica and dbt**, and ships with an AI assistant, a built-in MCP server and a data quality engine.

---

## 🆕 What's New (v0.15)

- **ETL tool lineage** — beyond SQL text: upload Kettle (.ktr/.kjb), SSIS (.dtsx), DataX (JSON), Informatica PowerCenter (XML) or dbt job files and get **column-level lineage inside ETL jobs** — which source tables flow through which transformation steps into which targets. Fully regression-tested against 355 real jobs from the official Kettle sample repository.
- **AI assistant** — a built-in AI agent with pluggable model endpoints (OpenAI-compatible APIs, DeepSeek, Qwen, …) that answers questions about analysis results and performs **lineage diagnosis**, helping you quickly figure out why a column's lineage breaks.
- **Built-in MCP server** — an MCP (Model Context Protocol) endpoint with token management, so AI clients such as Claude Desktop or Cursor can query column-level lineage directly through MCP and bring lineage into your AI workflow.
- **Data quality engine** — built-in quality rules (accuracy, validity, …) bound to assets via policies, with scheduled runs, asynchronous execution and Webhook alerts; row-level sampling, reference dataset comparison and custom checks, compatible with 14+ dialects.
- **Deep T-SQL enhancements** — a systematic overhaul for SQL Server procedures: all six EXEC forms, INSERT..EXEC, cursor lifecycles, TRY/CATCH, PIVOT/UNPIVOT, table-valued functions (TVF), SELECT INTO / temp tables and more now produce precise lineage.

---

## Repository Guide

| What | Link |
|------|------|
| 📦 Download the build | [GitHub Releases](https://github.com/estherdata2026/esther-official-resources/releases) · [Gitee releases](https://gitee.com/esther2026/esther-official-resources/releases) |
| 📚 Product manual (Chinese) | [docs/product-manual.zh-CN.md](docs/product-manual.zh-CN.md) |
| 📚 Product manual (English) | [docs/product-manual.en.md](docs/product-manual.en.md) |
| 🐛 Report a bug | [GitHub Issues](https://github.com/estherdata2026/esther-official-resources/issues) (please use an issue template) · [Gitee Issues](https://gitee.com/esther2026/esther-official-resources/issues) |
| 💡 Feature requests | [GitHub Issues](https://github.com/estherdata2026/esther-official-resources/issues) · [Gitee Issues](https://gitee.com/esther2026/esther-official-resources/issues) |
| 📮 Contact us | [estherdata@163.com](mailto:estherdata@163.com) (business / licensing) |

> **Note**: this repository distributes program builds and product documentation only. Product source code is not hosted here.

---

## Quick Start

Deploy in four steps (Windows preview build):

```text
1. Download the zip package from Releases and extract it anywhere
2. Request a free license (3 months): run esther-cli.exe license fingerprint, email
   the fingerprint to receive the esther.lic file, and place it in the program directory
3. Double-click start_esther.bat to start the server
4. Open http://localhost:8000 in your browser and sign in
```

- A valid license is required to start the server; new users can apply for a **free 3-month license**. Licenses are machine-bound — request a new one when migrating to another server.
- On first startup an administrator account **admin** (initial password admin123) is created automatically — keep it safe after signing in.
- On first visit to the lineage page, a demo SQL is analyzed automatically so you can see the lineage graph immediately.
- To change the port, set the `ESTHER_PORT` environment variable and restart.

See the [product manual](docs/product-manual.en.md) for full deployment, licensing and feature documentation.

---

## Key Capabilities

- **Column-level lineage** — field-level data flow tracking with 9 edge types: direct, transform, aggregation, filter, join and more.
- **30 SQL dialects** — covers mainstream databases (MySQL, PostgreSQL, Oracle, SQL Server, Snowflake, H2, …) and Chinese domestic databases (Dameng, Kingbase, Oscar, HighGo, GBase, OceanBase, GaussDB, …), with automatic dialect detection.
- **ETL job parsing** — direct parsing of five ETL tool formats (Kettle, SSIS, DataX, Informatica PowerCenter, dbt); transformation steps and column-level lineage merged into one graph.
- **Interactive lineage queries** — server-side traversal by direction / depth / type filters / endpoint tracing; one-click table-level vs. column-level views; full-path highlighting on selection; a spreadsheet "Table" view with filtering, sorting, pagination and CSV export.
- **Deep procedure parsing** — PL/SQL, PL/pgSQL, T-SQL and MySQL procedures, including local variables, cursors, control flow (IF/LOOP), triggers (NEW/OLD) and cross-procedure calls; T-SQL keeps getting deep enhancements.
- **Metadata extraction pipelines** — dual sources (data source connections + SQL script packages); diff-based merging automatically adds, updates and removes assets (dropped source tables leave no residue); run logs, error summaries and lineage snapshots.
- **Impact analysis** — graph-backed upstream tracing and downstream impact queries: find out which reports a column change would affect.
- **Data quality engine** — quality rules bound to assets via policies, with scheduled runs, Webhook alerts, row-level sampling and reference dataset checks.
- **AI assistant & MCP integration** — built-in AI Q&A / lineage diagnosis with pluggable models; a built-in MCP server for AI clients such as Claude and Cursor to query lineage directly.
- **Three ways to use** — Web UI (with a built-in developer API docs portal), REST API (with Swagger docs) and command-line CLI.

---

## Repository Layout

```text
esther-official-resources/
├── README.md                        # Home (Chinese)
├── README.en.md                     # Home (English)
├── docs/
│   ├── product-manual.zh-CN.md      # Product manual (Chinese)
│   └── product-manual.en.md         # Product manual (English)
└── .github/
    └── ISSUE_TEMPLATE/              # Bug report / feedback templates
```

---

## Support & Feedback

- For bugs, please open an issue on [GitHub](https://github.com/estherdata2026/esther-official-resources/issues) or [Gitee](https://gitee.com/esther2026/esther-official-resources/issues) and include the **SQL text, dialect, expected vs. actual result** as prompted by the template.
- For licensing & business inquiries, email 📮 [estherdata@163.com](mailto:estherdata@163.com), or leave a message via an issue.

---

**Esther** — The Wise Guardian of Your Data | 您的数据智慧管家
