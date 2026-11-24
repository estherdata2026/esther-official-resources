# 3. Quick Start: Your First Lineage Analysis


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
