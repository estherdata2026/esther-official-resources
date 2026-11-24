# 9. CLI 参考


命令行工具位于程序目录（Windows 为 `esther-cli.exe`），适合脚本化批量分析。

### 9.1 分析 SQL

```bash
# 直接分析一段 SQL，输出表格
esther-cli.exe analyze --dialect mysql --sql "SELECT a.id FROM orders a"

# 自动检测方言，分析 SQL 文件，输出 JSON（重定向保存）
esther-cli.exe analyze --dialect auto --file query.sql --format json > result.json

# 指定项目与图数据库路径
esther-cli.exe analyze -d postgres -s "SELECT * FROM t" --project etl --db-path esther.db
```

| 参数 | 说明 |
|------|------|
| `--dialect / -d` | 方言标识（必填），支持 `auto` 及所有别名 |
| `--sql / -s` | SQL 文本 |
| `--file / -f` | SQL 文件路径 |
| `--format` | `json` 或 `table`（默认） |
| `--project / -p` | 项目名，默认 `default` |
| `--db-path` | 图数据库文件路径 |
| `--config / -c` | 配置文件路径，默认读取同目录 `esther.toml` |

### 9.2 其他命令

```bash
esther-cli.exe dialects list        # 列出全部方言与别名
esther-cli.exe project create myprj # 创建项目
esther-cli.exe project list         # 项目列表
esther-cli.exe project export myprj # 导出项目内全部分析结果为 JSON
esther-cli.exe license status       # 查看授权状态
esther-cli.exe license fingerprint  # 输出本机授权指纹
```

### 9.3 退出码

| 退出码 | 含义 |
|--------|------|
| `0` | 成功 |
| `1` | 分析成功但有解析警告；`license status` 检测到授权被拦截时同样返回 |
| `2` | 错误（方言不存在 / 文件缺失 / 格式错误等） |
| `3` | 授权拦截（未授权或授权已过期） |

---
