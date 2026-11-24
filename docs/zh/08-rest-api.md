# 8. REST API 参考


所有接口的交互式文档（Swagger UI）位于：`http://<服务器IP>:8000/docs`

### 8.1 分析接口

接口：`POST /api/analyze`

**请求体**

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

| 字段 | 必填 | 说明 |
|------|------|------|
| `sql` | ✅ | SQL 文本，可包含多条语句、DDL、存储过程 |
| `dialect` | — | 方言标识，默认 `auto`（自动检测） |
| `project` | — | 项目名，默认 `default` |
| `table_schemas` | — | 表结构字典，键为表名/FQN，用于 `SELECT *` 展开与列消歧 |

**响应（节选）**

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

节点 `type` 为节点类型（COLUMN / PARAMETER / VARIABLE / LITERAL …），边 `type` 取值见 [4.5 边类型](#45-读懂血缘图)。

### 8.2 血缘查询接口（需登录 Token）

| 接口 | 用途 |
|------|------|
| `POST /api/v1/lineage/query` | 血缘视图查询：方向 / 深度 / 类型过滤 / 端点追溯，服务端遍历 |
| `GET /api/v1/lineage/snapshots` | 血缘快照列表 |
| `GET /api/v1/lineage/snapshots/{id}` | 快照详情 |
| `GET /api/v1/lineage/batches` | 批量解析批次列表 |
| `GET /api/v1/lineage/batches/{id}` | 批次详情 |
| `POST /api/v1/lineage/upload-batch` | REST 批量解析 SQL 脚本包（需有效授权） |

**`POST /api/v1/lineage/query` 请求体**

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

| 字段 | 取值 | 说明 |
|------|------|------|
| `analysis_id` / `datasource_id` | 二选一 | 分析级查询（分析页 / 沙箱结果）或数据源级查询（可附 `schema_name`） |
| `view.start` | `{type, id}` 或省略 | 起点（容器或列）；**省略 = 整图视图**（配合类型过滤做整图筛选） |
| `view.direction` | `upstream` / `downstream` / `both` | 遍历方向，默认 `both` |
| `view.depth` | `-1` 或正整数 | 遍历深度，`-1` 为不限（默认）；深度无硬性上限 |
| `view.show_node_types` / `view.show_edge_types` | 类型白名单或省略 | 只显示所列的节点 / 边类型；省略 = 不过滤 |
| `view.endpoints` | `upstream` / `downstream` / 省略 | 端点追溯：只返回起点 + 最上游来源 / 最终去向 |
| `view.show_isolated` | 布尔，默认 `false` | 整图视图下是否保留没有任何血缘关系的孤立节点 |

**响应（节选）**：`nodes` / `edges`（结构与 [8.1](#81-分析接口) 一致）+ `stats`（`result_nodes` / `result_edges` / `result_containers` 等）、`truncated`（结果是否被截断）、`max_depth`（起点沿该方向实际可达的最大深度）。

> 迁移说明：旧版的 `GET /api/v1/lineage/{fqn}` 与 `GET /api/v1/lineage/manifest` 接口已由 `POST /api/v1/lineage/query` 取代，原有集成请改用新接口。

### 8.3 采集任务接口（需登录 Token）

| 接口 | 用途 |
|------|------|
| `POST /api/v1/pipelines/upload-zip` | 上传 SQL 脚本包，返回 `zipFileId` |
| `POST /api/v1/pipelines` | 创建采集任务（数据源与脚本包至少其一） |
| `GET /api/v1/pipelines`、`GET /api/v1/pipelines/{id}` | 任务列表 / 详情 |
| `PUT /api/v1/pipelines/{id}` | 编辑任务（数据源 / 脚本包 / 方言 / 调度等） |
| `POST /api/v1/pipelines/{id}/run` | 立即运行（同数据源互斥） |
| `GET /api/v1/pipelines/{id}/runs` | 运行历史（含变更摘要、血缘产出、错误计数） |
| `GET /api/v1/pipelines/{id}/runs/{run_id}/log` | 查看运行日志（`…/log/download` 下载） |
| `POST /api/v1/pipelines/{id}/runs/{run_id}/orphans/adopt` | 孤儿表归入资产库 |

### 8.4 其他接口

| 接口 | 用途 |
|------|------|
| `GET /api/dialects` | 全部可用方言及别名（常用方言排前） |
| `POST /api/v1/auth/login` | 登录，获取 JWT Token |
| `/api/v1/datasources` 系列 | 数据源配置、连接测试、元数据同步（同步按差异合并自动增删改资产） |
| `GET /api/v1/projects`、`POST /api/v1/projects` | 项目列表 / 创建 |
| `GET /api/v1/auth/assets/{fqn}/children` | 资产下级结构列表 |

> ⚠️ `POST /api/analyze` 未启用鉴权，请仅在内网使用；`/api/v1/*` 需要先登录换取 Token；`POST /api/v1/lineage/upload-batch`（批量生成血缘）需要有效授权。

---
