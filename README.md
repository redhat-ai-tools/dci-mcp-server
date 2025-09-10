# DCI MCP Server

This project provides a Model Context Protocol (MCP) server adapted for the [DCI API](https://doc.distributed-ci.io/dci-control-server/docs/API/).
It allows AI models to interact with [DCI](https://doc.distributed-ci.io/) for comprehensive data extraction about DCI jobs, components, topics and files.

## Features

- 🚀 **FastAPI**: Built on a modern, fast web framework
- 🤖 **MCP**: Implements the Model Context Protocol for AI integration
- 🔍 **Comprehensive DCI API**: Full access to DCI components, jobs, files, pipelines, products, teams, and topics
- 🔧 **Smart PR Detection**: Advanced PR build finder that analyzes job URLs and metadata
- 🔐 **DCI Integration**: Native DCI API support with authentication
- 📝 **Easy Configuration**: Support for .env files for simple setup
- ✅ **Code Quality**: Comprehensive pre-commit checks and linting

## Installation

```bash
# Clone the repository
git clone https://github.com/redhat-ai-tools/dci-mcp-server
cd dci-mcp-server

# Install dependencies
uv sync

# Activate virtual environment
source .venv/bin/activate
```

### Configuration

The server supports multiple ways to configure DCI authentication:

Copy the example file and customize it:

```bash
cp env.example .env
# Edit .env with your DCI credentials
```

Example `.env` file:

```bash
# Method 1: API Key Authentication
DCI_CLIENT_ID=<client_type>/<client_id>
DCI_API_SECRET=<api_secret>

# Method 2: User ID/Password (alternative to API key)
# DCI_LOGIN=foo
# DCI_PASSWORD=bar
```

### MCP Configuration

#### Cursor IDE (stdio transport)

Add to your `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "dci": {
      "command": "uv",
      "args": ["run", "/path/to/dci-mcp-server/.venv/bin/python", "/path/to/dci-mcp-server/main.py"],
      "description": "MCP server for DCI integration"
    }
  }
}
```

#### Web-based Integration (SSE transport)

For web applications or services that need HTTP-based communication:

```json
{
  "mcpServers": {
    "dci": {
      "url": "http://0.0.0.0:8000/sse/",
      "description": "MCP server for DCI integration with direct SSE",
      "env": {
        "MCP_TRANSPORT": "sse"
      }
    }
  }
}
```

**SSE Endpoint**: `http://0.0.0.0:8000/sse/`

> **Note**: Make sure to start the SSE server separately with `MCP_TRANSPORT=sse uv run main.py` before using this configuration.

## Prompts

You can then use [prompts](PROMPTS.md) to explore the DCI data.

There are also parameterized prompts defined in the MCP server:

- `/dci/rca <job id>` conducts a Root Cause Analysis of the problem in the job. Storing the downloaded files under `/tmp/dci/<job id>` and generating a report at `/tmp/dci/rca-<job id>.md`.
- `/dci/weekly <team name/id or remoteci name/id>` conducts a report for the last 7 days stored at `/tmp/dci`.
- `/dci/biweekly <team name/id or remoteci name/id>` conducts a report for the last 14 days stored at `/tmp/dci`.

## Available Tools exposed by the MCP server

The server provides tools for interacting with DCI API components:

### Component Tools

- `query_dci_components(query, limit, offset, sort, fields)`: Query components with advanced query language and pagination

### Date Tools

- `today()`: Returns today's date in YYYY-MM-DD format

### Job Tools

- `search_dci_jobs(query, sort, limit, offset, fields)`: Search jobs with advanced query language and pagination

### File Tools

- `download_dci_file(job_id, file_id, output_path)`: Download a file to local path


## Code Quality Checks

The project includes comprehensive code quality checks:

#### Manual Checks
```bash
# Run all checks
bash scripts/run-checks.sh

# Or run individual checks
./.venv/bin/python -m black --check .
./.venv/bin/python -m isort --check-only .
./.venv/bin/python -m ruff check .
./.venv/bin/python -m mypy mcp_server/
./.venv/bin/python -m bandit -r .
```

#### Pre-commit Hooks (Optional)
```bash
# Install pre-commit hooks
./.venv/bin/python -m pre_commit install

# Run pre-commit on all files
./.venv/bin/python -m pre_commit run --all-files
```

## Development

### Project Structure

```
mcp_server/
├── config.py             # Configuration and authentication
├── main.py               # Server entry point
├── services/             # DCI API services
│   ├── dci_base_service.py
│   ├── dci_component_service.py
│   ├── dci_job_service.py
│   ├── dci_file_service.py
│   ├── dci_log_service.py
│   ├── dci_pipeline_service.py
│   ├── dci_product_service.py
│   ├── dci_team_service.py
│   ├── dci_remoteci_service.py
│   └── dci_topic_service.py
├── prompts/              # Templatized prompts
│   └── prompts.py
├── tools/                # MCP tools
│   ├── component_tools.py
│   ├── date_tools.py
│   ├── job_tools.py
│   ├── file_tools.py
│   └── log_tools.py
└── utils/                # Utility functions
    └── http_client.py
```

### Adding New Tools

1. Create a new service in `mcp_server/services/` if needed
2. Create a new tool file in `mcp_server/tools/`
3. Register the tools in `mcp_server/main.py`
4. Update this README with documentation
