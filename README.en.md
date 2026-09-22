# Esther · SQL Data Lineage Analysis Platform

**English** | [简体中文](README.md)

**The Wise Guardian of Your Data**

**Esther** is an enterprise SQL data lineage analysis platform. It extracts **column-level** data flow relationships directly from SQL text, supports 28 SQL dialects (including Chinese domestic databases such as Dameng, Kingbase and OceanBase), and provides an interactive lineage graph with upstream tracing and downstream impact analysis.

---

## Repository Guide

| What | Link |
|------|------|
| 📦 Download the trial build | [GitHub Releases](https://github.com/estherdata2026/esther-official-resources/releases) · [Gitee releases](https://gitee.com/esther2026/esther-official-resources/releases) |
| 📚 Product manual (Chinese) | [docs/product-manual.zh-CN.md](docs/product-manual.zh-CN.md) |
| 📚 Product manual (English) | [docs/product-manual.en.md](docs/product-manual.en.md) |
| 🐛 Report a bug | [GitHub Issues](https://github.com/estherdata2026/esther-official-resources/issues) (please use an issue template) · [Gitee Issues](https://gitee.com/esther2026/esther-official-resources/issues) |
| 💡 Feature requests | [GitHub Issues](https://github.com/estherdata2026/esther-official-resources/issues) · [Gitee Issues](https://gitee.com/esther2026/esther-official-resources/issues) |
| 📮 Contact us | [estherdata@163.com](mailto:estherdata@163.com) (business / licensing) |

> **Note**: this repository distributes trial builds and product documentation only. Product source code is not hosted here.

---

## Trial Quick Start

Deploy in three steps (Windows preview build):

```text
1. Download the zip package from Releases and extract it anywhere
2. Double-click start_esther.bat to start the server
3. Open http://localhost:8000 in your browser
```

- On first visit to the lineage page, a demo SQL is analyzed automatically so you can see the lineage graph immediately.
- To change the port, set the `ESTHER_PORT` environment variable and restart.
- When the trial expires, run `esther-cli.exe license fingerprint` and email the fingerprint to [estherdata@163.com](mailto:estherdata@163.com) to obtain a license file (`esther.lic`).

See the [product manual](docs/product-manual.en.md) for full deployment, licensing and feature documentation.

---

## Key Capabilities

- **Column-level lineage** — field-level data flow tracking with 9 edge types: direct, transform, aggregation, filter, join and more.
- **28 SQL dialects** — covers mainstream databases (MySQL, PostgreSQL, Oracle, SQL Server, …) and Chinese domestic databases (Dameng, Kingbase, Oscar, HighGo, GBase, OceanBase, …), with automatic dialect detection.
- **Deep procedure parsing** — PL/SQL, PL/pgSQL, T-SQL and MySQL procedures, including local variables, cursors, control flow (IF/LOOP), triggers (NEW/OLD) and cross-procedure calls.
- **Impact analysis** — graph-backed upstream tracing and downstream impact queries: find out which reports a column change would affect.
- **Three ways to use** — Web UI, REST API (with Swagger docs) and command-line CLI.

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
