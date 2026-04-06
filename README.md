# OpenViking Plugins

English | [简体中文](./README_zh.md)

Memory auto-recall and auto-capture for hook-enabled CLI agents, powered by [OpenViking](https://github.com/volcengine/OpenViking).

> **Claude Code** is fully supported. More agents are under active development.

## What It Does

OpenViking Plugins gives your CLI agent **persistent, cross-session, semantic memory** — completely transparently:

- **Auto-Recall** — Before every user message, relevant memories are silently injected into the agent's context
- **Auto-Capture** — After every response, new knowledge is automatically extracted and stored
- **MCP Tools** — On-demand memory operations (search, store, delete, health check) when the agent or user needs explicit control

No extra tool calls, no manual bookmarks. The agent simply *remembers*.

## Supported Agents

| Agent | Status | Integration |
|-------|--------|-------------|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | Supported | Hooks + MCP Server |
| More coming soon | In Development | — |

## Architecture

```
┌─────────────────────────────────┐
│          CLI Agent              │
│  (Claude Code, ...)             │
└──────┬──────────────────┬───────┘
       │                  │
  User Prompt Hook     Stop Hook
       │                  │
 ┌─────▼──────┐    ┌──────▼──────┐
 │ Auto-Recall│    │Auto-Capture │
 │            │    │             │
 │ Semantic   │    │ Extract new │
 │ search &   │    │ knowledge & │
 │ inject     │    │ store       │
 └─────┬──────┘    └──────┬──────┘
       │                  │
       │  ┌────────────┐  │
       └─►│ OpenViking │◄─┘
          │   Server   │
          └────────────┘
```

## Quick Start

### 1. Install & Start OpenViking

```bash
pip install openviking

# macOS alternative
brew install pipx && pipx install openviking
```

Create the config file `~/.openviking/ov.conf` (Can override the default path via the environment variable `OPENVIKING_CONFIG_FILE`):

```json
{
  "server": { "host": "127.0.0.1", "port": 1933 },
  "storage": {
    "workspace": "~/.openviking/data",
    "vectordb": { "backend": "local" },
    "agfs": { "backend": "local", "port": 1833 }
  },
  "embedding": {
    "dense": {
      "provider": "volcengine",
      "api_key": "<your-api-key>",
      "model": "doubao-embedding-vision-251215",
      "api_base": "https://ark.cn-beijing.volces.com/api/v3",
      "dimension": 1024,
      "input": "multimodal"
    }
  },
  "vlm": {
    "provider": "volcengine",
    "api_key": "<your-api-key>",
    "model": "doubao-seed-2-0-pro-260215",
    "api_base": "https://ark.cn-beijing.volces.com/api/v3"
  }
}
```

For Windows workspace paths, use `/` instead of `\`, for example `D:/.openviking/data`.

Start the server:

```bash
openviking-server
```

### 2. Install the Plugin (Claude Code)

```bash
/plugin marketplace add Castor6/openviking-plugins
/plugin install claude-code-memory-plugin@openviking-plugin
```

Then start a new Claude session — the plugin bootstraps itself automatically.

## Available Plugins

| Plugin | Description | Docs |
|--------|-------------|------|
| `claude-code-memory-plugin` | Long-term semantic memory for Claude Code | [README](./plugins/claude-code-memory-plugin/README.md) |

## How It Works

### Auto-Recall

1. User submits a prompt → the agent's pre-prompt hook fires
2. The plugin semantically searches OpenViking for relevant memories
3. Top-ranked results are injected into the agent's context as a system message
4. The agent sees `<relevant-memories>` transparently — no tool call needed

### Auto-Capture

1. The agent finishes responding → the post-response hook fires
2. The plugin reads the conversation transcript incrementally (only new turns)
3. New knowledge is extracted via OpenViking's AI-powered entity extraction
4. Memories are stored in the vector DB for future recall

### MCP Tools (Explicit)

When you need manual control, the MCP server exposes:

- `memory_recall` — Semantic search across all stored memories
- `memory_store` — Manually persist text as structured memories
- `memory_forget` — Delete memories by URI or search query
- `memory_health` — Check OpenViking server status

## Comparison with Built-in Memory

| Feature | Built-in (e.g. MEMORY.md) | OpenViking Plugin |
|---------|---------------------------|-------------------|
| Storage | Flat files | Vector DB + structured extraction |
| Search | Full context load | Semantic similarity |
| Scope | Per-project | Cross-project, cross-session |
| Capacity | Limited by context window | Unlimited |
| Extraction | Manual | AI-powered |

## Configuration

All configuration lives in `~/.openviking/ov.conf`. Override the path with:

```bash
export OPENVIKING_CONFIG_FILE="~/custom/path/ov.conf"
```

See individual plugin READMEs for agent-specific settings.

## Project Structure

```
openviking-plugins/
├── plugins/
│   └── claude-code-memory-plugin/   # Claude Code integration
│       ├── hooks/                   # Hook definitions
│       ├── scripts/                 # Hook scripts & utilities
│       ├── servers/                 # Compiled MCP server
│       ├── src/                     # MCP server source (TypeScript)
│       └── README.md               # Detailed plugin docs
├── .claude-plugin/
│   └── marketplace.json             # Plugin marketplace registry
├── README.md                        # This file
├── README_zh.md                     # 中文文档
└── LICENSE
```

## License

[Apache-2.0](./LICENSE) — same as [OpenViking](https://github.com/volcengine/OpenViking).
