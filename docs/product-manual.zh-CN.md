# Esther 产品说明书

**Esther · 数据血缘分析平台 | The Wise Guardian of Your Data**

| | |
|------|------|
| 适用版本 | 0.10.0-preview（Windows 预览版） |
| 文档更新日期 | 2026-08-23 |
| 英文版 | [product-manual.en.md](product-manual.en.md) |

---

## 目录

1. [产品概述](#1-产品概述)
2. [安装与授权](#2-安装与授权)
3. [快速上手：第一次血缘分析](#3-快速上手第一次血缘分析)
4. [数据血缘分析（核心功能）](#4-数据血缘分析核心功能)
   - [4.1 什么是列级血缘](#41-什么是列级血缘)
   - [4.2 三种分析入口](#42-三种分析入口)
   - [4.3 支持的 SQL 方言](#43-支持的-sql-方言)
   - [4.4 可分析的 SQL 范围](#44-可分析的-sql-范围)
   - [4.5 读懂血缘图](#45-读懂血缘图)
   - [4.6 图交互操作](#46-图交互操作)
   - [4.7 上游溯源与下游影响分析](#47-上游溯源与下游影响分析)
   - [4.8 元数据辅助：让血缘更准](#48-元数据辅助让血缘更准)
5. [REST API 参考](#5-rest-api-参考)
6. [CLI 参考](#6-cli-参考)
7. [配置说明](#7-配置说明)
8. [常见问题与已知限制](#8-常见问题与已知限制)
9. [获取帮助](#9-获取帮助)

---

## 1. 产品概述

**Esther** 是一个企业级 SQL 数据血缘分析平台。它直接解析 SQL 文本，自动提取**列级别**的数据流转关系——某张报表字段的数据从哪些表哪些列来、中间经历了什么计算、又流向了哪里——并以交互式血缘图谱呈现。

### 1.1 解决什么问题

| 场景 | 没有血缘时 | 有了 Esther |
|------|-----------|------------|
| **变更影响分析** | 改一个底层表字段，不知道会炸哪些下游报表 | 在图上选中该列，一眼看到全部下游链路 |
| **数据治理** | 表、视图、存储过程的关系散落在成千上万行 SQL 里 | 自动解析 SQL，生成完整血缘图谱 |
| **合规审计 / 数据溯源** | 被问"这个对外字段的数据从哪来"时人工翻代码 | 上游溯源功能逐层回溯到数据源头 |
| **数仓迁移 / 重构评估** | 人工评估工作量，容易遗漏 | 基于血缘图统计受影响对象，量化评估 |

### 1.2 三种使用形态

- **Web UI** — 浏览器中粘贴 SQL 即时出图，交互式浏览血缘
- **REST API** — 提供完整的 OpenAPI/Swagger 文档，便于与自有系统集成
- **命令行 CLI** — 脚本化批量分析，输出 JSON 或表格

### 1.3 典型能力一览

- 列级血缘：精确到字段，区分直传 / 转换 / 聚合 / 过滤 / 关联等 9 种边类型
- 28 种 SQL 方言，覆盖主流数据库与达梦、人大金仓等信创数据库，支持方言自动检测
- 存储过程深度解析：PL/SQL、PL/pgSQL、T-SQL、MySQL 存储过程，含变量、游标、控制流、触发器
- 上游溯源 / 下游影响分析：基于图数据库的多跳遍历
- 血缘结果导出：PNG 图片、JSON、CSV

---

## 2. 安装与授权

### 2.1 部署三步（Windows 预览版）

1. 从 [Releases](https://github.com/estherdata2026/esther-official-resources/releases) 下载 zip 压缩包，解压到目标服务器任意目录（绿色免安装）。
2. 双击 `start_esther.bat`（或命令行运行）启动服务。
3. 浏览器打开 `http://<服务器IP>:8000` 即可使用。

**修改端口**：设置环境变量 `ESTHER_PORT` 后重启服务。

### 2.2 授权（License）

- **试用模式**：首次运行自带试用期，试用期内功能完整，到期后服务停止。
- **转正式授权**：
  1. 命令行执行 `esther-cli.exe license fingerprint`，获取本机指纹；
  2. 将指纹发送给厂商，换取授权文件 `esther.lic`；
  3. 将 `esther.lic` 放到程序目录，重启服务即进入注册模式。
- **查看授权状态**：`esther-cli.exe license status`。

### 2.3 运行数据与账号

- 运行时数据自动创建在程序目录的 `data/` 下（SQLite + Kuzu 图数据库），无需额外安装数据库。
- 默认管理员账号见首次启动日志。
- 换服务器 / 换机器需要重新申请授权（指纹与机器绑定）。

### 2.4 安全提示

- 程序为 Nuitka 原生编译产物，部分杀毒软件可能误报，请加入白名单；依赖已静态链接，无需安装 VC++ 运行库。
- `POST /api/analyze` 接口当前未启用鉴权，请将服务部署在**内网**环境中使用；`/api/v1/*` 系列接口需要登录后携带 Token 访问。

---

## 3. 快速上手：第一次血缘分析

打开浏览器进入系统后：

1. 在左侧导航点击 **数据血缘**，进入 SQL 血缘分析页。
   页面首次加载时会**自动运行一段演示 SQL** 并渲染血缘图谱——你打开就能看到效果。
2. 在 SQL 编辑器中粘贴自己的 SQL（支持多条语句、存储过程、CREATE 语句等）。
3. 在方言下拉框选择对应数据库；不确定时保持 **自动检测**。
4. 点击 **分析** 按钮（快捷键 `Ctrl + Enter`）。
5. 查看结果：
   - 顶部统计栏：**节点数 / 边数 / 警告数** 及识别出的方言；
   - **血缘图**标签页：交互式图谱（点击、高亮、导出 PNG，见 [4.5](#45-读懂血缘图) 与 [4.6](#46-图交互操作)）；
   - **JSON**标签页：原始分析结果，可直接对照 API 文档；
   - 底部警告面板：解析警告可点击定位到 SQL 对应位置。

> 提示：分析页是"即席分析沙箱"，结果不落库。要让血缘持久保存并支持跨对象溯源，请使用元数据采集任务（见 [4.7](#47-上游溯源与下游影响分析)）。

---

## 4. 数据血缘分析（核心功能）

### 4.1 什么是列级血缘

给定一段 SQL，Esther 提取出"**哪个列的数据流向了哪个列**"以及"**中间发生了什么**"。例如：

```sql
INSERT INTO report (id, total)
SELECT order_id, price * qty
FROM orders;
```

Esther 生成如下血缘：

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

- `orders.order_id` **原样**写入 `report.id` → 灰色实线 `DIRECT`
- `report.total` 由 `price * qty` **计算**而来 → 紫色虚线 `TRANSFORM`，连线上的 `𝑓` 徽章悬停即可查看表达式

表级血缘（表 → 表）由列血缘自动归组得到，无需额外配置。

### 4.2 三种分析入口

| | Web UI 分析页 | REST API | CLI |
|---|---|---|---|
| **入口** | 导航"数据血缘" | `POST /api/analyze` | `esther-cli.exe analyze` |
| **适合** | 交互式探索、看图 | 与自有系统集成、批量调用 | 脚本化、CI、离线分析 |
| **输入** | SQL 文本 + 方言 | SQL + 方言 + 可选表结构 | SQL 文本或 `.sql` 文件 |
| **输出** | 交互式图谱 + JSON | JSON（同 UI 的 JSON 标签页） | JSON 或命令行表格 |
| **详见** | 第 3 章 | 第 5 章 | 第 6 章 |

三者共享同一套解析引擎，结果结构完全一致。

### 4.3 支持的 SQL 方言

Esther 支持 **28 种** SQL 方言。常用别名（如 `pg`、`mysql`）也可作为方言标识传入。

**国际主流数据库**

| 数据库 | 方言标识 | 常用别名 |
|--------|----------|----------|
| MySQL | `mysql` | `mariadb`、`tidb` |
| PostgreSQL | `postgres` | `postgresql`、`pg`、`opengauss` |
| Oracle | `oracle` | `oracledb` |
| SQL Server (T-SQL) | `tsql` | `sqlserver`、`mssql` |
| IBM DB2 | `db2` | |
| SAP HANA | `hana` | `saphana` |
| Sybase ASE | `sybase` | `ase` |
| Informix | `informix` | |
| Teradata | `teradata` | `td` |
| SQLite | `sqlite` | `sqlite3` |
| ClickHouse | `clickhouse` | `ch` |
| Snowflake | `snowflake` | |
| BigQuery | `bigquery` | |
| Presto | `presto` | `prestodb`、`prestosql` |
| Trino | `trino` | |
| Hive | `hive` | `hiveql` |
| Spark / Databricks | `spark` | `sparksql`、`databricks`、`delta` |
| Amazon Redshift | `redshift` | |
| Apache Doris | `doris` | `apachedoris` |
| StarRocks | `starrocks` | |
| DuckDB | `duckdb` | |

**信创 / 国产数据库**

| 数据库 | 方言标识 | 常用别名 |
|--------|----------|----------|
| 达梦 DM | `dameng` | `dm`、`dm7`、`dm8` |
| 人大金仓 KingbaseES | `kingbase` | `kingbasees`、`kes` |
| 神通 Oscar | `oscar` | `shentong` |
| 瀚高 HighGo | `highgo` | |
| 南大通用 GBase | `gbase` | `gbase8s`、`gbase8a` |
| OceanBase（MySQL 模式） | `oceanbase` | `ob`、`oceanbase_mysql` |
| OceanBase（Oracle 模式） | `oceanbase_oracle` | |
| GaussDB | `gaussdb` | |

**方言自动检测**：方言选择"自动检测"（或传 `auto`）时，Esther 先按 SQL 关键词指纹缩小候选，再尝试逐个解析、以"解析错误最少者"当选。对混合来源的历史 SQL 脚本尤其有用。

> 各版本实际可用方言以产品内方言下拉框或 `GET /api/dialects` 接口返回为准。

### 4.4 可分析的 SQL 范围

**查询与 DML**

- `SELECT`（含子查询、相关子查询、递归 CTE、窗口函数）
- `INSERT INTO … SELECT`、`UPDATE … SET`、`DELETE`、`MERGE`
- `UNION / UNION ALL`
- Hive/Spark 的 `LATERAL VIEW`、ClickHouse 的 `ARRAY JOIN`

**建表与视图**

- `CREATE TABLE AS SELECT`（CTAS）、`CREATE VIEW AS SELECT`、物化视图
- DDL 中的表结构会被自动提取，用于列消歧与 `SELECT *` 展开
- `ALTER TABLE … RENAME` 产生"重命名"血缘

**CTE 与临时表**

- `WITH` 子句（CTE）单独成卡，逐层展开
- 临时表（T-SQL `#temp`、`CREATE TEMP TABLE`）正常追踪

**存储过程 / 函数 / 包**（深度解析）

- 支持 Oracle PL/SQL（匿名块、命名过程、包）、PostgreSQL PL/pgSQL、SQL Server T-SQL（含 `GO` 批处理）、MySQL 存储过程
- 追踪**局部变量**赋值流转、**游标**（`DECLARE` / `FETCH INTO`）、`IF / LOOP / FOR` 控制流中的 DML
- **跨过程调用**：`CALL proc_b(...)` 的数据流会接续传递
- **动态 SQL 检测**：`EXECUTE IMMEDIATE`、`sp_executesql`、`PREPARE` 等动态构造的 SQL 无法静态展开，Esther 会给出 `DYNAMIC_SQL` 警告并将相关节点置信度标记为 0.3

**触发器**

- 触发器采用三层模型：①源表 → 触发器函数的**绑定边**（`⚡` 徽章，悬停显示 `BEFORE INSERT FOR EACH ROW` 等事件信息）；②函数体内 `NEW.col / OLD.col` → 目标表的**数据流**；③体内 DML 的 WHERE 条件 → 目标容器的**过滤边**

**文件与导入导出**

- `COPY … TO / FROM '文件'` 产生表 ↔ 文件节点的血缘，文件卡片上标注路径、格式（CSV/PARQUET/JSON 等）、分隔符与导入/导出方向

### 4.5 读懂血缘图

#### 卡片（容器）

血缘图按"容器"归组：每张表 / 视图 / CTE / 存储过程等渲染为一张卡片，卡片内列出其列、参数、变量。不同容器类型用不同颜色区分：

| 卡片类型 | 示例 | 颜色 |
|----------|------|------|
| 表 TABLE | `orders` | 蓝色 |
| 视图 VIEW | `user_view` | 青色 |
| 物化视图 | `mv_sales` | 深青色 |
| CTE | `recent` | 绿色 |
| 子查询 | `(SELECT …) s` | 紫色 |
| 临时表 | `#temp` | 灰蓝色 |
| 存储过程 / 函数 | `sp_update` | 深紫色 |
| 文件 | `orders.dat` | 棕色 |
| 游标 / 匿名块 / 事件 | `c`、`BEGIN…END`、`CREATE EVENT` | 橙 / 灰 / 青 |

#### 卡片内条目

每个列条目可携带：数据类型、主键（PK）/ 外键（FK）/ 分区键（P）/ 分布键（D）徽章、参数方向（IN/OUT/INOUT/RETURN）、**置信度百分比**，以及计算列的 `𝑓` 表达式徽章。

#### 边类型（9 种）

| 类型 | 含义 | 线型 | 徽章（悬停内容） |
|------|------|------|------------------|
| `DIRECT` | 值原样传递 | 灰色实线 | — |
| `TRANSFORM` | 一对一计算（`price * qty`） | 紫色虚线 | `𝑓`（表达式） |
| `AGGREGATION` | 多对一聚合（`SUM(x) GROUP BY …`） | 橙色双线 | `Σ`（表达式 + 分组） |
| `UNION` | 多源合并 | 青色波纹线 | — |
| `JOIN` | 两表关联条件（列 ↔ 列） | 绿色点划线 | `⋈`（`ON` 条件） |
| `FILTER` | 过滤条件（列 → 表卡片） | 红色点线 | `⊳`（`WHERE` 条件） |
| `RENAME` | 表/列重命名 | 紫色长虚线 | 文字"重命名" |
| `TRIGGER` | 触发绑定（表卡片 → 触发器卡片） | 琥珀色虚线 | `⚡`（事件信息） |
| `BRANCH` | 分支执行路径 | 黄色 | — |

页面上可打开 **图例** 弹层随时对照线型与颜色。

#### 结构关系（细线）

除数据流外，外键（FOREIGN_KEY）、同义词（SYNONYM_OF）、分区（PARTITION_OF）等**结构性关系**以半透明细线渲染，与数据流边明显区分。

#### 置信度

| 值 | 含义 |
|----|------|
| 100% | 来自 DDL 定义或显式列清单，确定 |
| 80% | 由 FROM 子句推断的表列 |
| 70% | 分支（IF/CASE）中的 SQL |
| 30% | 涉及动态 SQL，静态分析无法展开 |
| 0% | 占位列名（如 `INSERT` 无列清单时） |

#### 警告（Warnings）

解析过程产生的提示按严重度分级，点击警告可**定位到 SQL 原文对应位置**：

| 警告码 | 含义 |
|--------|------|
| `PARSE_ERROR` | 某条语句解析失败，该语句血缘可能缺失 |
| `RESOLVE_ERROR` | 某条语句未能生成完整血缘 |
| `DYNAMIC_SQL` | 检测到动态 SQL，相关血缘置信度 0.3 |
| `GOTO_UNCERTAINTY` | 过程中存在 GOTO，无法静态确定某 DML 是否执行，已保守标记 |

### 4.6 图交互操作

| 操作 | 效果 |
|------|------|
| **单击某个列 / 卡片标题** | 沿该列**双向**高亮整条链路——上游来路 + 下游去向全部点亮，其余淡出 |
| **单击某条连线或徽章** | SQL 编辑器自动选中并高亮产生该边的 SQL 片段 |
| **悬停徽章**（`𝑓` `Σ` `⊳` `⋈` `⚡`） | 弹出表达式 / 条件 / 事件详情 |
| **拖拽卡片 / 缩放 / 小地图** | 自由调整布局；自动布局为从左到右分层 |
| **导出 PNG** | 将当前血缘图导出为图片，可直接贴入文档 |
| **JSON 标签页** | 查看与 API 完全一致的结构化结果 |

### 4.7 上游溯源与下游影响分析

这是血缘图谱的持久化形态，解决"改这一列，影响谁"的问题：

1. **建立持久血缘**：在"数据源"页配置数据库连接并运行**元数据采集任务**。采集完成后，系统会自动解析库中的视图 / 物化视图定义（以及过程、触发器），将血缘结果保存进内置图数据库（Kuzu），并生成**血缘快照**（含节点/边数量、版本、受影响对象清单）。
2. **查看**：进入任意数据资产的详情页，打开 **血缘图谱** 标签：
   - **完整血缘** — 上游 + 下游全图
   - **上游溯源** — 只看数据从哪里来，逐层回溯到源头
   - **变更历史 / 快照** — 对比不同采集版本之间的血缘差异
3. **多跳遍历**：溯源/影响分析默认不限制深度（系统上限 30 跳），可通过 API 的 `depth` 参数控制。

> 分析页（第 3 章）与持久血缘的关系：分析页是即席沙箱，不写入图数据库；采集任务产生的血缘才持久保存并参与跨对象溯源。

### 4.8 元数据辅助：让血缘更准

三种方式为解析器提供表结构信息，提升准确率：

| 方式 | 操作 | 解决的问题 |
|------|------|-----------|
| **内联表结构** | 调用 API 时在 `table_schemas` 中传入表结构 | `SELECT *` 按真实列展开；同名列消歧 |
| **粘贴 DDL** | SQL 中直接包含 `CREATE TABLE` 语句 | 同上，且无需额外参数 |
| **元数据采集** | 配置数据源后运行采集任务 | 全库规模化解析，视图定义穿透（视图 → 视图 → 表逐层展开） |

示例——`table_schemas` 用法见 [5.1](#51-分析接口)。

---

## 5. REST API 参考

所有接口的交互式文档（Swagger UI）位于：`http://<服务器IP>:8000/docs`

### 5.1 分析接口

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
| `project` — | — | 项目名，默认 `default` |
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

### 5.2 血缘查询接口（需登录 Token）

| 接口 | 用途 |
|------|------|
| `GET /api/v1/lineage/{fqn}` | 按 FQN 查询某资产的上游/下游血缘 |
| `GET /api/v1/lineage/manifest` | 按数据源 + 层级（库/模式/表/列）参数化查询 |
| `GET /api/v1/lineage/snapshots` | 血缘快照列表 |
| `GET /api/v1/lineage/snapshots/{id}` | 快照详情 |

**常用查询参数**

| 参数 | 取值 | 说明 |
|------|------|------|
| `direction` | `upstream` / `downstream` / `both` | 遍历方向，默认双向 |
| `depth` | `-1`~`30` | 遍历深度，`-1` 为不限（默认），上限 30 |
| `asset_type` | `database` / `schema` / `table` / `view` / `column` | 资产层级 |
| `project` | 项目名 | 限定项目范围 |

### 5.3 其他接口

| 接口 | 用途 |
|------|------|
| `GET /api/dialects` | 全部可用方言及别名（常用方言排前） |
| `POST /api/v1/auth/login` | 登录，获取 JWT Token |
| `/api/v1/datasources` 系列 | 数据源配置、连接测试、元数据同步 |

> ⚠️ `POST /api/analyze` 未启用鉴权，请仅在内网使用；`/api/v1/*` 需要先登录换取 Token。

---

## 6. CLI 参考

命令行工具位于程序目录（Windows 为 `esther-cli.exe`），适合脚本化批量分析。

### 6.1 分析 SQL

```bash
# 直接分析一段 SQL，输出表格
esther-cli.exe analyze --dialect mysql --sql "SELECT a.id FROM orders a"

# 分析 SQL 文件，输出 JSON（重定向保存）
esther-cli.exe analyze --dialect auto --file query.sql --format json > result.json

# 指定项目与图数据库路径
esther-cli.exe analyze -d postgres -s "SELECT * FROM t" --project etl --db-path esther.db
```

| 参数 | 说明 |
|------|------|
| `--dialect / -d` | 方言标识，支持 `auto` 及所有别名 |
| `--sql / -s` | SQL 文本 |
| `--file / -f` | SQL 文件路径 |
| `--format` | `json` 或 `table`（默认） |
| `--project / -p` | 项目名，默认 `default` |
| `--db-path` | 图数据库文件路径 |
| `--config / -c` | 配置文件路径，默认读取同目录 `esther.toml` |

### 6.2 其他命令

```bash
esther-cli.exe dialects list        # 列出全部方言与别名
esther-cli.exe project create myprj # 创建项目
esther-cli.exe project list         # 项目列表
esther-cli.exe project export myprj # 导出项目内全部分析结果为 JSON
esther-cli.exe license status       # 查看授权状态
esther-cli.exe license fingerprint  # 输出本机授权指纹
```

### 6.3 退出码

| 退出码 | 含义 |
|--------|------|
| `0` | 分析成功，无警告 |
| `1` | 分析成功，但产生了解析警告 |
| `2` | 错误（方言不存在 / 文件缺失 / 格式错误等） |
| `3` | 授权拦截（试用期到期或未授权） |

---

## 7. 配置说明

配置文件为程序目录下的 `esther.toml`。

### 7.1 存储后端

```toml
[storage]
# 默认 kuzu：嵌入式图数据库，零部署；可选 neo4j：服务端图数据库
backend = "kuzu"

[storage.kuzu]
db_path = "./esther.db"

# 切换 Neo4j（可选）
# [storage.neo4j]
# uri      = "bolt://localhost:7688"
# user     = "neo4j"
# password = "secret"
# database = "esther"
```

### 7.2 服务端口

通过环境变量 `ESTHER_PORT` 修改（默认 `8000`），修改后需重启服务。

### 7.3 血缘验证智能体（可选，进阶）

`esther.toml` 中预留了**双模型血缘验证**配置：由两个独立大模型分别"预测血缘"与"仲裁正确性"，用于对血缘结果做抽样质检。需要在环境变量中注入对应 API Key（`DEEPSEEK_API_KEY`、`DASHSCOPE_API_KEY`），普通使用无需配置。

---

## 8. 常见问题与已知限制

**Q：选择"自动检测"识别出的方言不对？**
A：混合方言的 SQL 脚本可能被误判，手动指定方言更可靠。可通过 `GET /api/dialects` 确认可用方言列表。

**Q：动态 SQL（`EXECUTE IMMEDIATE` / `sp_executesql`）能分析吗？**
A：静态分析无法展开运行时才拼出的 SQL。Esther 会检测到动态 SQL 并给出 `DYNAMIC_SQL` 警告，相关血缘节点置信度标注为 30%，不会被静默丢弃。

**Q：分析页的结果刷新后没了？**
A：分析页是即席沙箱，不持久化。需要长期保存、跨对象溯源请使用元数据采集任务（[4.7](#47-上游溯源与下游影响分析)）。

**已知限制**

| 限制 | 说明 |
|------|------|
| 溯源深度上限 | 图遍历最多 30 跳（覆盖绝大多数真实链路） |
| 自动布局层数 | 单图最多 12 层，超大图建议按资产粒度查看 |
| `POST /api/analyze` 无鉴权 | 请内网部署 |
| GOTO 静态不确定性 | 过程内 GOTO 跳过的 DML 以 `GOTO_UNCERTAINTY` 警告 + 保守标记呈现 |
| 杀软误报 | Nuitka 编译产物可能被个别杀软误报，请加白名单 |

---

## 9. 获取帮助

- **Bug 反馈 / 功能建议**：请到 [Issues](https://github.com/estherdata2026/esther-official-resources/issues) 按模板提交。为了快速定位问题，请尽量提供：
  1. SQL **原文**（完整语句，可脱敏但请保持结构）
  2. 方言标识
  3. 期望的血缘结果 vs 实际结果（截图或 JSON）
  4. 版本号（`esther-cli.exe license status` 或启动日志中查看）
- **授权与商务**：请通过 Issue 留言或联系发布方。

---

*Esther — The Wise Guardian of Your Data | 您的数据智慧管家*
