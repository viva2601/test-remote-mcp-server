# test-remote-mcp-server: Expense Tracker MCP Server

A remote [Model Context Protocol](https://modelcontextprotocol.io) (MCP) server built with [FastMCP](https://gofastmcp.com). It gives any MCP-compatible AI client (such as Claude) tools to record and analyse personal expenses, backed by an asynchronous SQLite database and served over HTTP.

## Features

- **Three tools** for managing expenses (add, list, summarise)
- **A resource** that returns the available expense categories as JSON
- **Asynchronous database access** with `aiosqlite`, so requests do not block each other
- **Remote HTTP transport**, so clients connect over a URL instead of launching a local process
- **Error handling:** every tool returns a structured error message instead of crashing (including a clear message when the database is read-only)

## Tools

| Tool | Arguments | Description |
|---|---|---|
| `add_expense` | `date`, `amount`, `category`, `subcategory` (optional), `note` (optional) | Adds an expense and returns its new `id` |
| `list_expenses` | `start_date`, `end_date` | Lists expenses in an inclusive date range, newest first |
| `summarize` | `start_date`, `end_date`, `category` (optional) | Returns the total amount and number of expenses per category, largest first |

Use ISO dates (`YYYY-MM-DD`) so that the date range filters work correctly.

## Resource

| URI | Description |
|---|---|
| `expense:///categories` | The expense categories as JSON, read from `categories.json` (food, transport, rent, shopping, bills, health, entertainment, other). If the file is missing, a built-in default list is returned |

## Project structure

```
.
├── main.py             # FastMCP server: database setup, tools, resource
├── categories.json     # Expense categories served by the resource
├── pyproject.toml      # Dependencies (fastmcp, aiosqlite)
└── README.md
```

The SQLite database (`expenses.db`) is created automatically when the server starts, so it should not be committed.

## Run locally

**Requirements:** Python 3.14+ (as set in `pyproject.toml`) and [uv](https://docs.astral.sh/uv/).

```bash
uv sync
uv run main.py
```

The server starts on `http://localhost:8000`, and the MCP endpoint is `http://localhost:8000/mcp`.

**Test it** with the MCP Inspector:
```bash
npx @modelcontextprotocol/inspector
```
Choose the Streamable HTTP transport and enter `http://localhost:8000/mcp`. Then call `add_expense`, `list_expenses`, and `summarize`.

**Connect a client:** add the server URL as a custom connector (or remote MCP server) in an MCP-compatible client, then ask for things like "Add a 250 rupee food expense for today" or "Summarise my spending this month by category".

## Deploy

The server runs over HTTP, so it can be deployed to any host that runs a Python web service, including FastMCP's own hosting. After deploying, use the public URL ending in `/mcp` in your client.

## Notes and limitations

- The database is stored in the system temp folder, so data may be lost when the server restarts or redeploys. For real use, point `DB_PATH` at a persistent location or switch to a hosted database.
- The server has no authentication, so do not expose it publicly with real data.
- Categories are not validated when adding an expense, and `categories.json` is informational only.

## Tech stack

Python, FastMCP, aiosqlite (SQLite), Model Context Protocol

## Author

**Vivaswan Tiwari**
[LinkedIn](https://www.linkedin.com/in/vivaswan-tiwari39853b140) · [GitHub](https://github.com/viva2601)
