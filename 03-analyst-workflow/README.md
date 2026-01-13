# Part 3: The Analyst Workflow with MCP Integration

> **Goal:** Orchestrate complex multi-step analysis using LangGraph and MCP tools.

---

## 🎯 What You'll Build

A **LangGraph workflow** that chains together multiple analysis steps using **MCP (Model Context Protocol) tools**:
1. **Fetch Stock** → Get current stock data via MCP tools
2. **Search News** → Find recent market news via MCP tools
3. **Retrieve Docs** → Query ingested PDFs via RAG
4. **Analyze** → Generate a comprehensive analysis

---

## 📋 Scenario

> "Build an analyst workflow for {COMPANY_NAME}: Fetch stock data → Search news → Retrieve internal docs → Analyze → Generate preliminary report."

**Track A Focus:** Analyze the Couche-Tard takeover: risk vs. opportunity  
**Track B Focus:** Analyze Sakura's government AI contracts: sustainable growth vs. bubble

---

## 🚀 Steps

### 1. Install Dependencies

```bash
cd 03-analyst-workflow
uv sync
```

### 2. Run the Workflow

```bash
uv run python analyst_agent.py
```

---

## 🏗️ Workflow Architecture

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐     ┌───────────────┐
│ User Query  │────▶│ Fetch Stock  │────▶│ Search News  │────▶│ Retrieve Docs │
│ {COMPANY}   │     │              │     │              │     │ (RAG)         │
└─────────────┘     └──────────────┘     └──────────────┘     └───────┬───────┘
                                                                      │
                                                                      ▼
                                                            ┌─────────────────┐
                                                            │ Analyze &       │
                                                            │ Summarize       │
                                                            └─────────────────┘
```

---

## ⚠️ Exercise Design

**The graph structure is 100% pre-built.** You focus ONLY on implementing the logic inside each node.

### What's Provided (DO NOT MODIFY)
- State schema (`AnalystState`)
- Graph definition with all nodes and edges
- Main execution loop

### What You Implement
- `fetch_stock_node()` – Connect to MCP server and call stock tools
- `search_news_node()` – Connect to MCP server and call news tools
- `retrieve_docs_node()` – Query HANA vector store
- `analyze_node()` – Craft prompt and call LLM

---

## 🏋️ Exercises

### Exercise 3a: `fetch_stock_node`
```python
async def fetch_stock_node(state: AnalystState) -> dict:
    # TODO: Connect to MCP server from 02-data-connector-mcp
    # TODO: Use get_stock_info tool with state["ticker"]
    # Return: {"stock_info": {...}}
```

### Exercise 3b: `search_news_node`
```python
async def search_news_node(state: AnalystState) -> dict:
    # TODO: Connect to MCP server from 02-data-connector-mcp
    # TODO: Use search_market_news tool with company name
    # Return: {"news_results": "..."}
```

### Exercise 3c: `retrieve_docs_node`
```python
def retrieve_docs_node(state: AnalystState) -> dict:
    # TODO: Query HANA vector store with state["query"]
    # Return: {"doc_context": "..."}
```

### Exercise 3d: `analyze_node`
```python
def analyze_node(state: AnalystState) -> dict:
    # TODO: Combine stock_info, news_results, doc_context
    # TODO: Craft a prompt and call the LLM
    # Return: {"analysis": "..."}
```

---

## 🔌 MCP Integration

This module integrates with the **Model Context Protocol (MCP)** to access tools from the `02-data-connector-mcp` module:

### MCP Tools Used
- **`get_stock_info(ticker)`** - Fetches current stock data using yfinance
- **`search_market_news(query, limit)`** - Searches recent news using Perplexity AI

### MCP Connection Pattern
```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client
from langchain_mcp_adapters.tools import load_mcp_tools

# Connect to MCP server
server_params = StdioServerParameters(
    command=sys.executable,
    args=[str(Path(__file__).parent.parent / "02-data-connector-mcp" / "mcp_server.py")],
)

async with stdio_client(server_params) as (read, write):
    async with ClientSession(read, write) as session:
        await session.initialize()
        tools = await load_mcp_tools(session)
        
        # Use tools
        stock_tool = next(tool for tool in tools if tool.name == "get_stock_info")
        result = await stock_tool.ainvoke({"ticker": "3778.T"})
```

---

## 💡 Key Concepts

### LangGraph State
```python
class AnalystState(TypedDict):
    company_name: str
    ticker: str
    query: str
    stock_info: dict
    news_results: str  # Changed to string for MCP integration
    doc_context: str
    analysis: str
```

### Node Functions
Each node receives the current state and returns updates:
```python
def my_node(state: AnalystState) -> dict:
    # Do work...
    return {"field_to_update": new_value}
```

### Graph Edges
```python
graph.add_edge("node_a", "node_b")  # A → B
graph.add_edge(START, "first_node")
graph.add_edge("last_node", END)
```

---

## ✅ Success Criteria

```
🔄 Starting analyst workflow for Sakura Internet (3778.T)
📊 Step 1: Fetching stock data...
   ✅ Price: ¥5,230 (▲2.3%)
📰 Step 2: Searching news...
   ✅ Found 5 relevant articles
📄 Step 3: Retrieving documents...
   ✅ Retrieved 5 relevant chunks
🧠 Step 4: Analyzing...

=== ANALYSIS RESULT ===
Based on the available data...
```

---

## ➡️ Next Step

Once your workflow produces analysis, proceed to **Part 4: The Deal Memo Generator**!
