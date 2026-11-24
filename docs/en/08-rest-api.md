# 8. REST API Reference


Interactive API documentation (Swagger UI): `http://<server-ip>:8000/docs`

### 8.1 Analysis endpoint

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

### 8.2 Lineage query endpoints (login token required)

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

**Response (excerpt)**: `nodes` / `edges` (same structure as [8.1](#81-analysis-endpoint)) plus `stats` (`result_nodes` / `result_edges` / `result_containers` …), `truncated` (whether the result was cut off) and `max_depth` (the actually reachable maximum depth from the start point).

> Migration note: the old `GET /api/v1/lineage/{fqn}` and `GET /api/v1/lineage/manifest` endpoints have been replaced by `POST /api/v1/lineage/query`; existing integrations should move to the new endpoint.

### 8.3 Extraction pipeline endpoints (login token required)

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

### 8.4 Other endpoints

| Endpoint | Purpose |
|----------|---------|
| `GET /api/dialects` | All available dialects and aliases (common ones first) |
| `POST /api/v1/auth/login` | Login to obtain a JWT token |
| `/api/v1/datasources` | Data source CRUD, connection testing, metadata sync (sync merges by diff, adding/updating/removing assets automatically) |
| `GET /api/v1/projects`, `POST /api/v1/projects` | Project list / creation |
| `GET /api/v1/auth/assets/{fqn}/children` | Child structure list of an asset |

> ⚠️ `POST /api/analyze` has no authentication — use it on internal networks only; `/api/v1/*` requires a login token; `POST /api/v1/lineage/upload-batch` (batch lineage generation) requires a valid license.

---
