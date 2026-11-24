# Esther · 数据血缘分析平台

[English](README.en.md) | **简体中文**

**The Wise Guardian of Your Data | 您的数据智慧管家**

**Esther** 是一个企业级 SQL 数据血缘分析平台：从 SQL 文本中自动提取**列级别**的数据流转关系，覆盖 30 种 SQL 方言（含达梦、人大金仓、OceanBase、GaussDB 等信创数据库），并提供可视化血缘图谱、方向/深度/类型过滤/端点追溯等交互式血缘查询、上游溯源与下游影响分析。此外还能直接解析 **Kettle、SSIS、DataX、Informatica、dbt** 等 ETL 工具的作业文件，内置 AI 智能体助手、MCP Server 与数据质量引擎。

---

## 🆕 最新特性亮点（v0.15）

- **ETL 工具血缘分析** — 不再局限于 SQL 文本：直接上传 Kettle（.ktr/.kjb）、SSIS（.dtsx）、DataX（JSON）、Informatica PowerCenter（XML）、dbt 作业文件，即可获得 ETL 作业内部的**列级血缘**——数据从哪个源表经过哪些转换步骤流向哪个目标表，一目了然。已用 Kettle 官方样例仓 355 个真实作业全量回归验证。
- **AI 智能体助手** — 平台内置 AI 助手：支持接入自定义大模型端点（OpenAI 兼容接口、DeepSeek、通义千问等），可对分析结果进行**智能问答与血缘诊断**，帮你快速定位"这一列的血缘为什么断了"。
- **内置 MCP Server** — 提供 MCP（Model Context Protocol）端点与访问令牌管理，Claude Desktop、Cursor 等 AI 客户端可通过 MCP 直接查询列级血缘，把血缘能力带进你的 AI 工作流。
- **数据质量引擎** — 内置准确性/有效性等质量规则，策略绑定资产、定时调度、异步执行、Webhook 告警；支持行级采样核查、参考数据集比对与自定义检查，14+ 方言兼容。
- **T-SQL 血缘深度增强** — 对 SQL Server 存储过程做了系统性攻坚：EXEC 六种形态、INSERT..EXEC、游标生命周期、TRY/CATCH、PIVOT/UNPIVOT、表值函数（TVF）、SELECT INTO/临时表等复杂语法均能产出精确血缘。

---

## 仓库导航

| 内容 | 链接 |
|------|------|
| 📦 下载安装包 | [GitHub Releases](https://github.com/estherdata2026/esther-official-resources/releases) · [Gitee 发行版](https://gitee.com/esther2026/esther-official-resources/releases) |
| 📚 产品说明书（中文） | [docs/product-manual.zh-CN.md](docs/product-manual.zh-CN.md) |
| 📚 产品说明书（英文） | [docs/product-manual.en.md](docs/product-manual.en.md) |
| 🐛 提交 Bug 反馈 | [GitHub Issues](https://github.com/estherdata2026/esther-official-resources/issues)（请选择问题模板） · [Gitee Issues](https://gitee.com/esther2026/esther-official-resources/issues) |
| 💡 功能建议 | [GitHub Issues](https://github.com/estherdata2026/esther-official-resources/issues) · [Gitee Issues](https://gitee.com/esther2026/esther-official-resources/issues) |
| 📮 联系我们 | [estherdata@163.com](mailto:estherdata@163.com)（商务合作 / 授权咨询） |

> **说明**：本仓库仅发布程序安装包与产品文档，产品源代码不在本仓库中。

---

## 快速上手

四步即可完成部署（Windows 预览版）：

```text
1. 从 Releases 页面下载 zip 压缩包，解压到任意目录
2. 申请免费授权（3 个月）：运行 esther-cli.exe license fingerprint 获取本机指纹，
   发送至邮箱换取授权文件 esther.lic，放入程序目录
3. 双击 start_esther.bat 启动服务
4. 浏览器打开 http://localhost:8000，使用管理员账号登录
```

- 启动需要有效授权；新用户可申请**免费 3 个月**的 license，授权与机器绑定，换服务器 / 换机器需重新申请。
- 首次启动自动创建管理员账号 **admin**（初始密码 admin123），登录后请妥善保管。
- 首次打开"数据血缘"页面时，系统会自动运行一段演示 SQL 并渲染血缘图谱，开箱即可体验。
- 修改端口：设置环境变量 `ESTHER_PORT` 后重启。

完整部署、授权与功能说明见[产品说明书](docs/product-manual.zh-CN.md)。

---

## 核心能力速览

- **列级血缘分析** — 精确到字段级别的数据流转追踪，区分直传、转换、聚合、过滤、关联等 9 种边类型。
- **30 种 SQL 方言** — 覆盖 MySQL、PostgreSQL、Oracle、SQL Server、Snowflake、H2 等主流数据库，以及达梦、人大金仓、神通、瀚高、GBase、OceanBase、GaussDB 等信创数据库；支持方言自动检测。
- **ETL 工具作业解析** — Kettle、SSIS、DataX、Informatica PowerCenter、dbt 五类 ETL 作业文件直接解析，转换步骤与列级血缘合成一张图谱。
- **交互式血缘查询** — 方向 / 深度 / 类型过滤 / 端点追溯由服务端遍历，表级 / 列级一键切换，选中即全路径高亮；"表格"视图支持筛选、排序、分页与导出 CSV。
- **存储过程深度解析** — 支持 PL/SQL、PL/pgSQL、T-SQL、MySQL 存储过程，含局部变量、游标、控制流（IF/LOOP）、触发器（NEW/OLD）与跨过程调用；T-SQL 场景持续深度增强。
- **元数据采集任务** — 数据源与 SQL 脚本包双来源，差异合并自动增删改资产（删表不残留），运行日志、错误摘要与血缘快照。
- **影响分析** — 基于图数据库的上游溯源与下游影响查询，评估"改这一列会影响哪些报表"。
- **数据质量引擎** — 质量规则 + 策略绑定资产，定时调度、Webhook 告警、行级采样与参考数据集核查。
- **AI 助手与 MCP 集成** — 内置 AI 智能问答 / 血缘诊断，可接入自定义大模型；内置 MCP Server 供 Claude、Cursor 等 AI 客户端直连查询血缘。
- **三种使用方式** — Web UI 可视化界面（内置开发者 API 文档门户）、REST API（Swagger 文档自带）、命令行 CLI。

---

## 目录结构

```text
esther-official-resources/
├── README.md                        # 仓库首页（中文）
├── README.en.md                     # 仓库首页（英文）
├── docs/
│   ├── product-manual.zh-CN.md      # 产品说明书（中文）
│   └── product-manual.en.md         # 产品说明书（英文）
└── .github/
    └── ISSUE_TEMPLATE/              # Bug 反馈 / 功能建议模板
```

---

## 支持与反馈

- 使用问题与 Bug：请提交 Issue（[GitHub](https://github.com/estherdata2026/esther-official-resources/issues) / [Gitee](https://gitee.com/esther2026/esther-official-resources/issues)），并尽量填写**SQL 文本、方言、期望结果与实际结果**，以便快速定位。
- 商务与授权咨询：欢迎联系 📮 [estherdata@163.com](mailto:estherdata@163.com)，或通过 Issue 留言。

---

**Esther** — The Wise Guardian of Your Data | 您的数据智慧管家
