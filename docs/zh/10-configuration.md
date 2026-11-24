# 10. 配置说明


配置文件为程序目录下的 `esther.toml`。

### 10.1 存储后端

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

### 10.2 服务端口

通过环境变量 `ESTHER_PORT` 修改（默认 `8000`），修改后需重启服务。

### 10.3 血缘验证智能体（可选，进阶）

`esther.toml` 中预留了**双模型血缘验证**配置：由两个独立大模型分别"预测血缘"与"仲裁正确性"，用于对血缘结果做抽样质检。需要在环境变量中注入对应 API Key（`DEEPSEEK_API_KEY`、`DASHSCOPE_API_KEY`），普通使用无需配置。

---
