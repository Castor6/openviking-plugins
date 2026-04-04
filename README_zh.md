# OpenViking Plugins

[English](./README.md) | 简体中文

为支持 Hook 的 CLI Agent 实现基于 [OpenViking](https://github.com/volcengine/OpenViking) 的记忆自动召回与捕获。

> **Claude Code** 已完整支持，更多 Agent 正在开发中。

## 功能概述

OpenViking Plugins 为你的 CLI Agent 提供**持久化、跨会话、语义化的长期记忆** — 完全透明无感：

- **自动召回** — 每次用户发送消息前，自动注入相关记忆到 Agent 上下文
- **自动捕获** — 每次回复结束后，自动提取新知识并存储
- **MCP 工具** — 按需进行记忆操作（搜索、存储、删除、健康检查）

无需额外的工具调用，无需手动标记。Agent 自然而然地「记住」一切。

## 支持的 Agent

| Agent | 状态 | 集成方式 |
|-------|------|----------|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | 已支持 | Hooks + MCP Server |
| 更多即将推出 | 开发中 | — |

## 架构

```
┌─────────────────────────────────┐
│          CLI Agent              │
│  (Claude Code, ...)             │
└──────┬──────────────────┬───────┘
       │                  │
  用户提交 Hook        停止 Hook
       │                  │
 ┌─────▼──────┐    ┌──────▼──────┐
 │  自动召回  │    │  自动捕获   │
 │            │    │             │
 │ 语义搜索   │    │ 提取新知识  │
 │ 并注入上下文│    │ 并存储      │
 └─────┬──────┘    └──────┬──────┘
       │                  │
       │  ┌────────────┐  │
       └─►│ OpenViking │◄─┘
          │   Server   │
          └────────────┘
```

## 快速开始

### 1. 安装并启动 OpenViking

```bash
pip install openviking

# macOS 可选方式
brew install pipx && pipx install openviking
```

创建配置文件 `~/.openviking/ov.conf`：

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
      "api_key": "<你的-API-Key>",
      "model": "doubao-embedding-vision-251215",
      "api_base": "https://ark.cn-beijing.volces.com/api/v3",
      "dimension": 1024,
      "input": "multimodal"
    }
  },
  "vlm": {
    "provider": "volcengine",
    "api_key": "<你的-API-Key>",
    "model": "doubao-seed-2-0-pro-260215",
    "api_base": "https://ark.cn-beijing.volces.com/api/v3"
  }
}
```

启动服务器：

```bash
openviking-server
```

### 2. 安装插件（Claude Code）

```bash
/plugin marketplace add Castor6/openviking-plugins
/plugin install claude-code-memory-plugin@openviking-plugin
```

然后启动一个新的 Claude 会话 — 插件会自动完成初始化。

## 可用插件

| 插件 | 说明 | 文档 |
|------|------|------|
| `claude-code-memory-plugin` | Claude Code 长期语义记忆 | [README](./plugins/claude-code-memory-plugin/README.md) |

## 工作原理

### 自动召回

1. 用户提交消息 → Agent 的预处理 Hook 触发
2. 插件对 OpenViking 进行语义搜索，查找相关记忆
3. 排名靠前的结果作为系统消息注入 Agent 上下文
4. Agent 透明地看到 `<relevant-memories>` — 无需任何工具调用

### 自动捕获

1. Agent 完成回复 → 后处理 Hook 触发
2. 插件增量读取对话记录（仅处理新增轮次）
3. 通过 OpenViking 的 AI 实体提取功能，从中抽取新知识
4. 记忆存入向量数据库，供未来召回使用

### MCP 工具（显式调用）

需要手动控制时，MCP 服务器提供以下工具：

- `memory_recall` — 语义搜索所有已存储的记忆
- `memory_store` — 手动将文本持久化为结构化记忆
- `memory_forget` — 按 URI 或查询删除记忆
- `memory_health` — 检查 OpenViking 服务器状态

## 与内置记忆的对比

| 特性 | 内置记忆（如 MEMORY.md） | OpenViking 插件 |
|------|--------------------------|-----------------|
| 存储 | 纯文本文件 | 向量数据库 + 结构化提取 |
| 搜索 | 全量加载到上下文 | 语义相似度搜索 |
| 范围 | 单项目 | 跨项目、跨会话 |
| 容量 | 受上下文窗口限制 | 无限制 |
| 提取 | 手动规则 | AI 驱动的实体提取 |

## 配置

所有配置存储在 `~/.openviking/ov.conf`。可通过环境变量覆盖路径：

```bash
export OPENVIKING_CONFIG_FILE="~/custom/path/ov.conf"
```

各 Agent 的特定设置请参阅对应插件的 README。

## 项目结构

```
openviking-plugins/
├── plugins/
│   └── claude-code-memory-plugin/   # Claude Code 集成
│       ├── hooks/                   # Hook 定义
│       ├── scripts/                 # Hook 脚本及工具
│       ├── servers/                 # 编译后的 MCP 服务器
│       ├── src/                     # MCP 服务器源码（TypeScript）
│       └── README.md               # 详细插件文档
├── .claude-plugin/
│   └── marketplace.json             # 插件市场注册信息
├── README.md                        # English
├── README_zh.md                     # 本文件
└── LICENSE
```

## 许可证

[Apache-2.0](./LICENSE) — 与 [OpenViking](https://github.com/volcengine/OpenViking) 一致。
