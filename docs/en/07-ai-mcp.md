# 7. AI Assistant & MCP


Esther ships a built-in **AI assistant** (a paid module) and an **MCP server**, making it possible to "ask the data" and "use lineage inside AI tools".

### 7.1 Prerequisite: connect a model (Model Access)

AI chat relies on a chat model endpoint. Configure it before first use:

1. Open **Model Access** in the left navigation;
2. Click **Create** and fill in the model name, base URL, API key and parameters (any OpenAI-compatible endpoint works: DeepSeek, Qwen, privately deployed models, …);
3. Click **Test** to verify connectivity, then bind it as the chat model under **Capability bindings**;
4. Enable the assistant with the **Assistant** toggle.

If no model is configured, the chat panel shows a "chat model not configured" notice with a link to Model Access; all existing platform features are unaffected.

### 7.2 AI chat (Q&A)

Click the **AI assistant** floating ball to open the chat panel:

- Ask in natural language, e.g. "What fields does the orders main table have? Who owns it?", "Which data sources did recent lineage analyses cover?", "How did the recent data quality checks go?";
- The assistant answers from the platform's own structured data — asset catalog, lineage, data quality results — with Markdown and **cited sources**, so every answer is traceable;
- Lineage mentioned in an answer can be **opened as an interactive lineage graph**;
- Conversation history and reply copying are supported.

### 7.3 AI lineage diagnosis

On the lineage analysis result page, click **AI Diagnosis**:

1. A deterministic **rule check** runs first (dangling references / missing suspects / low confidence, …);
2. The AI then analyzes the lineage in depth and drafts review conclusions;
3. The result is an **AI lineage diagnosis report** with one of three verdicts — no lineage errors found / suspected issues found / uncertain — plus an issue list with evidence.

> AI diagnosis is model-generated and for reference only; verify manually before acting on it.

### 7.4 MCP server: connect AI clients directly

Esther has a built-in MCP (Model Context Protocol) server at `http://<host>:8000/api/mcp` (streamable HTTP). Any MCP client that supports streamable HTTP (Claude Code, Cursor, …) can connect.

**Step 1: create an MCP API token**

Open **MCP API Token** in the left navigation → **Create Token** → name it (e.g. "Claude Code (data team)") and pick an expiry (30 / 90 / 365 days or never). The token is shown in plain text only once — save it immediately; it can be revoked at any time.

**Step 2: connect from your AI client**

For example, add this to Claude Code's MCP settings:

```json
{
  "mcpServers": {
    "esther": {
      "url": "http://<host>:8000/api/mcp",
      "headers": { "Authorization": "Bearer estm_..." }
    }
  }
}
```

**Available tools**

| Tool | Description |
|------|-------------|
| `query_lineage` | Lineage query by analysis ID / datasource / node, with direction and depth |
| `query_assets` | Asset catalog search |
| `query_quality` | Data quality results |
| `search_glossary` | Business glossary search |
| `query_sql` | Restricted queries over the platform's own data |
| `list_dialects` | List supported SQL dialects |

> Security & gating: MCP queries only touch the platform's own data (assets / lineage / quality / glossary) and never connect directly to your registered data source databases; every tool call (including failures) is audit-logged; MCP is gated by the license feature `assistant`.

---
