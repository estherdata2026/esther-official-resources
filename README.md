# Esther · 数据血缘分析平台

[English](README.en.md) | **简体中文**

**The Wise Guardian of Your Data | 您的数据智慧管家**

**Esther** 是一个企业级 SQL 数据血缘分析平台：从 SQL 文本中自动提取**列级别**的数据流转关系，覆盖 28 种 SQL 方言（含达梦、人大金仓、OceanBase 等信创数据库），并提供可视化血缘图谱、上游溯源与下游影响分析。

---

## 仓库导航

| 内容 | 链接 |
|------|------|
| 📦 下载试用版 | [Releases 发布页](https://github.com/estherdata2026/esther-official-resources/releases) |
| 📚 产品说明书（中文） | [docs/product-manual.zh-CN.md](docs/product-manual.zh-CN.md) |
| 📚 产品说明书（英文） | [docs/product-manual.en.md](docs/product-manual.en.md) |
| 🐛 提交 Bug 反馈 | [Issues](https://github.com/estherdata2026/esther-official-resources/issues)（请选择问题模板） |
| 💡 功能建议 | [Issues](https://github.com/estherdata2026/esther-official-resources/issues) |

> **说明**：本仓库仅发布试用程序与产品文档，产品源代码不在本仓库中。

---

## 试用版快速上手

三步即可完成部署（Windows 预览版）：

```text
1. 从 Releases 页面下载 zip 压缩包，解压到任意目录
2. 双击 start_esther.bat 启动服务
3. 浏览器打开 http://localhost:8000
```

- 首次打开"数据血缘"页面时，系统会自动运行一段演示 SQL 并渲染血缘图谱，开箱即可体验。
- 修改端口：设置环境变量 `ESTHER_PORT` 后重启。
- 试用期到期后，运行 `esther-cli.exe license fingerprint` 获取本机指纹，联系我们换取授权文件 `esther.lic`。

完整部署、授权与功能说明见[产品说明书](docs/product-manual.zh-CN.md)。

---

## 核心能力速览

- **列级血缘分析** — 精确到字段级别的数据流转追踪，区分直传、转换、聚合、过滤、关联等 9 种边类型。
- **28 种 SQL 方言** — 覆盖 MySQL、PostgreSQL、Oracle、SQL Server 等主流数据库，以及达梦、人大金仓、神通、瀚高、GBase、OceanBase 等信创数据库；支持方言自动检测。
- **存储过程深度解析** — 支持 PL/SQL、PL/pgSQL、T-SQL、MySQL 存储过程，含局部变量、游标、控制流（IF/LOOP）、触发器（NEW/OLD）与跨过程调用。
- **影响分析** — 基于图数据库的上游溯源与下游影响查询，评估"改这一列会影响哪些报表"。
- **三种使用方式** — Web UI 可视化界面、REST API（Swagger 文档自带）、命令行 CLI。

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

- 使用问题与 Bug：请提交 [Issue](https://github.com/estherdata2026/esther-official-resources/issues)，并按模板填写**SQL 文本、方言、期望结果与实际结果**，以便快速定位。
- 商务与授权咨询：请通过 Issue 留言或联系发布方。

---

**Esther** — The Wise Guardian of Your Data | 您的数据智慧管家
