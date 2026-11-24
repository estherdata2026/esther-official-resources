# 10. Configuration


The configuration file is `esther.toml` in the program directory.

### 10.1 Storage backend

```toml
[storage]
# Default: kuzu — embedded graph database, zero deployment. Optional: neo4j.
backend = "kuzu"

[storage.kuzu]
db_path = "./esther.db"

# Switch to Neo4j (optional)
# [storage.neo4j]
# uri      = "bolt://localhost:7688"
# user     = "neo4j"
# password = "secret"
# database = "esther"
```

### 10.2 Server port

Change via the `ESTHER_PORT` environment variable (default `8000`); restart to apply.

### 10.3 Lineage verifier agent (optional, advanced)

`esther.toml` ships with a reserved **dual-model lineage verification** configuration: two independent LLMs act as predictor and arbiter to spot-check lineage results. It requires the corresponding API keys in environment variables (`DEEPSEEK_API_KEY`, `DASHSCOPE_API_KEY`); regular usage does not need it.

---
