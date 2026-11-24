# 6. ETL 工具血缘分析


SQL 之外，Esther 还能直接解析主流 ETL 工具的作业文件，产出与 SQL 血缘**同构**的列级血缘图谱——同一套图谱交互、同一条查询 API、同一个图数据库。数据"在 ETL 工具里做了什么"从此不再是黑盒。

### 6.1 支持的 ETL 工具

| ETL 工具 | 作业文件 | 说明 |
|----------|----------|------|
| Kettle (Pentaho) | `.ktr`（转换）/ `.kjb`（作业） | 步骤 SQL 按对应方言解析并合成血缘 |
| SSIS | `.dtsx` 包 | |
| DataX | 作业 `.json` 配置 | 自定义 UDF 可注册为 transform / passthrough |
| Informatica PowerCenter | 导出 XML（或无扩展名导出文件） | 无扩展名文件需在工具列表中手动指定 |
| dbt | `manifest.json`（`target/` 编译产物） | 可选附加 `catalog.json` 联合解析 |

- 文件大小上限 **20MB**。
- 工具类型默认**自动识别**（按文件内容判断），也可手动指定。
- 已用 Kettle 官方样例仓 355 个真实作业全量回归验证。

### 6.2 使用步骤（Web UI）

1. 左侧导航进入 **数据血缘** 页的 **ETL 模式**（菜单提供直达入口，或访问 `/lineage?mode=etl`）。
2. 选择 ETL 工具（默认自动识别），**上传文件**或直接**粘贴内容**。
3. 点击 **分析 ETL**。
4. 查看结果：
   - **步骤条带**：作业内步骤清单（名称 / 类型 / 类别 / 方言 / 连接 / 目标表），点击步骤可高亮它在图谱中的血缘边；
   - **血缘图 / 表格 / JSON**：与 SQL 分析页完全同构的三个视图；
   - **方言报告**：每个连接最终采用了哪个方言、如何判定（显式指定 / 连接类型推断 / URL 推断 / 兜底默认）。
5. 无法识别方言的连接：页面会列出这些连接，逐个选择方言后**带方言重新分析**，其步骤 SQL 即纳入血缘。

> 高级选项：可为无法识别的连接预先指定方言、设置兜底方言；dbt 分析可附加 `catalog.json` 获得更完整的列类型与注释信息。

### 6.3 REST API

| 接口 | 说明 |
|------|------|
| `POST /api/v1/etl/analyze` | JSON body：`content`（文件内容）+ `path`（文件名）+ `tool`（默认 `auto`），可选 `default_dialect` / `connection_dialects` / `transformer_udfs` / `catalog_json` |
| `POST /api/v1/etl/analyze/upload` | multipart 文件上传，参数以 form 字段传递；dbt 可附加 `catalog` 文件 |

返回结构与 SQL 分析接口同构（`nodes` / `edges` / `warnings` / `metadata_tree`），另含 `tool`、`job`（作业与步骤概要）与 `dialect_report`。ETL 血缘同样写入图数据库（分析 ID 带 `etl:` 前缀），可通过[血缘查询接口](#82-血缘查询接口需登录-token)追溯。

> ETL 血缘分析受 license `etl` 特性门禁；未开通时返回 403 并附申请指引，不会静默失败。

### 6.4 已知边界

- 步骤中的 SQL 遵循对应 SQL 方言的解析能力（见 [4.3](#43-支持的-sql-方言)）；
- 连接无法识别方言时，其步骤 SQL 不参与血缘，直到补齐方言重新分析；
- 已禁用（disabled）的 hop 不产生血缘；空流步骤会给出显式告警。

---
