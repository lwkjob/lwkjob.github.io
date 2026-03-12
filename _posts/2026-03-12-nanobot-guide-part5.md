---
layout: post
title: "nanobot实战指南：从配置到部署的完整工作流"
date: 2026-03-12 04:00:00 +0800
categories: [AI, nanobot, tutorial]
tags: [nanobot, deployment, configuration, practical guide]
---

# nanobot实战指南：从配置到部署的完整工作流

## 前言

在前面的文章中，我们深入探讨了nanobot的架构设计、核心模块和关键代码实现。现在，让我们将理论知识转化为实践，通过完整的配置和部署流程，让您真正掌握nanobot的使用方法。

## 1. 环境准备与安装

### 1.1 系统要求
nanobot基于Python 3.11+开发，需要确保您的系统满足以下要求：
- Python 3.11 或更高版本
- pip包管理器
- 基本的网络连接（用于安装依赖）

### 1.2 安装方式选择

nanobot提供了三种安装方式，每种都有其适用场景：

#### 方式一：从源码安装（推荐用于开发）
```bash
git clone https://github.com/HKUDS/nanobot.git
cd nanobot
pip install -e .
```

这种方式的优势在于可以随时修改源码并立即生效，非常适合开发者进行二次开发。

#### 方式二：使用uv工具安装（推荐用于生产）
```bash
uv tool install nanobot-ai
```

uv是一个现代化的Python包管理工具，安装速度快且依赖解析更可靠。

#### 方式三：从PyPI安装（最简单）
```bash
pip install nanobot-ai
```

这是最简单的方式，适合快速体验nanobot的基本功能。

## 2. 初始化配置

### 2.1 运行初始化向导
```bash
nanobot onboard
```

这个命令会自动创建必要的配置文件和工作目录结构。初始化后，您会在`~/.nanobot/`目录下看到以下文件：
- `config.json` - 主配置文件
- `workspace/` - 工作空间目录
- 其他运行时文件

### 2.2 配置LLM提供商

nanobot支持多种LLM提供商，包括OpenRouter、Anthropic、OpenAI等。以OpenRouter为例：

```json
{
  "providers": {
    "openrouter": {
      "apiKey": "sk-or-v1-xxx"
    }
  },
  "agents": {
    "defaults": {
      "model": "anthropic/claude-opus-4-5",
      "provider": "openrouter"
    }
  }
}
```

### 2.3 配置安全设置

安全性是nanobot的重要考虑因素。默认情况下，nanobot会拒绝所有未授权的访问：

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN",
      "allowFrom": ["YOUR_USER_ID"]
    }
  },
  "tools": {
    "restrictToWorkspace": true
  }
}
```

`allowFrom`字段控制谁可以与您的bot交互，`restrictToWorkspace`确保所有工具操作都限制在工作空间目录内。

## 3. 聊天渠道集成

### 3.1 Telegram集成

Telegram是最常用的集成渠道，配置相对简单：

1. 在Telegram中搜索@BotFather
2. 使用`/newbot`命令创建新bot
3. 复制生成的token
4. 在配置文件中添加：

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "123456:ABCDEF",
      "allowFrom": ["123456789"]
    }
  }
}
```

### 3.2 WhatsApp集成

WhatsApp集成需要额外的步骤：

```bash
# 首先链接设备
nanobot channels login
# 扫描QR码完成配对

# 然后在配置文件中启用
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": ["+1234567890"]
    }
  }
}
```

### 3.3 多渠道同时运行

nanobot支持同时运行多个渠道，您可以在同一个配置文件中启用多个渠道：

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "telegram_token"
    },
    "discord": {
      "enabled": true,
      "token": "discord_token"
    },
    "whatsapp": {
      "enabled": true
    }
  }
}
```

## 4. 启动和运行

### 4.1 启动网关

配置完成后，启动nanobot网关：

```bash
nanobot gateway
```

网关会监听所有启用的渠道，并处理传入的消息。

### 4.2 本地CLI测试

在启动网关的同时，您也可以使用CLI进行本地测试：

```bash
nanobot agent -m "Hello, nanobot!"
```

### 4.3 系统服务部署

对于生产环境，建议将nanobot作为系统服务运行：

```ini
# ~/.config/systemd/user/nanobot-gateway.service
[Unit]
Description=Nanobot Gateway
After=network.target

[Service]
Type=simple
ExecStart=%h/.local/bin/nanobot gateway
Restart=always
RestartSec=10
NoNewPrivileges=yes
ProtectSystem=strict
ReadWritePaths=%h

[Install]
WantedBy=default.target
```

然后启用服务：
```bash
systemctl --user daemon-reload
systemctl --user enable --now nanobot-gateway
```

## 5. 高级配置和优化

### 5.1 多实例运行

nanobot支持多实例运行，这对于需要为不同用途或不同用户运行独立bot的场景非常有用：

```bash
# 实例A - Telegram bot
nanobot gateway --config ~/.nanobot-telegram/config.json

# 实例B - Discord bot
nanobot gateway --config ~/.nanobot-discord/config.json
```

每个实例都有独立的配置、工作空间和运行时数据。

### 5.2 MCP（Model Context Protocol）集成

nanobot支持MCP协议，可以集成外部工具服务器：

```json
{
  "tools": {
    "mcpServers": {
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
      },
      "my-remote-mcp": {
        "url": "https://example.com/mcp/",
        "headers": {
          "Authorization": "Bearer xxxxx"
        }
      }
    }
  }
}
```

### 5.3 性能优化

对于高负载场景，可以进行以下优化：

1. **使用vLLM本地模型**：
```json
{
  "providers": {
    "vllm": {
      "apiKey": "dummy",
      "apiBase": "http://localhost:8000/v1"
    }
  }
}
```

2. **调整工具超时设置**：
```json
{
  "tools": {
    "mcpServers": {
      "my-slow-server": {
        "url": "https://example.com/mcp/",
        "toolTimeout": 120
      }
    }
  }
}
```

## 6. 监控和维护

### 6.1 日志查看

```bash
# 查看系统服务日志
journalctl --user -u nanobot-gateway -f

# CLI模式下显示日志
nanobot agent --logs
```

### 6.2 心跳任务

nanobot支持定期执行心跳任务，通过编辑`~/.nanobot/workspace/HEARTBEAT.md`文件：

```markdown
## Periodic Tasks

- [ ] Check weather forecast and send a summary
- [ ] Scan inbox for urgent emails
```

### 6.3 更新和维护

```bash
# 更新到最新版本
pip install -U nanobot-ai

# 重启服务
systemctl --user restart nanobot-gateway
```

## 7. 故障排除

### 7.1 常见问题

1. **渠道不响应**：检查`allowFrom`配置和API token
2. **内存占用过高**：启用`tools.restrictToWorkspace`限制
3. **连接超时**：调整`toolTimeout`设置

### 7.2 调试技巧

```bash
# 启用详细日志
nanobot gateway --verbose

# 检查配置状态
nanobot status
```

## 结语

通过本系列文章，我们从nanobot的基础概念、架构设计、核心代码到实际部署，全面了解了这个超轻量级AI助手的方方面面。nanobot的设计哲学是"少即是多"，用最少的代码实现最大的功能价值。

无论您是想要快速搭建一个个人AI助手，还是希望深入研究AI代理的实现原理，nanobot都是一个绝佳的选择。它的简洁性和可扩展性使其既适合初学者学习，也适合高级开发者进行定制开发。

在AI代理技术快速发展的今天，理解这些底层实现原理将帮助您更好地把握技术趋势，为未来的创新奠定坚实基础。