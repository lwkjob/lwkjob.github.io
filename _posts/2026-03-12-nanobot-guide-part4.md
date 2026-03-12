---
layout: post
title: "nanobot模块设计详解：Channels、Providers与Skills系统"
date: 2026-03-12 04:00:00 +0800
categories: [AI, nanobot, architecture]
tags: [nanobot, channels, providers, skills, modular design]
---

# nanobot模块设计详解：Channels、Providers与Skills系统

## 引言

在前几篇文章中，我们探讨了nanobot的入门使用和核心架构。本文将深入分析nanobot的三大核心模块：Channels（渠道集成）、Providers（LLM提供商）和Skills（技能系统）。这些模块共同构成了nanobot的扩展性基础。

## Channels模块设计

### 架构概览

nanobot的Channels模块负责与各种聊天平台的集成。根据官方文档，nanobot支持以下渠道：
- Telegram
- Discord  
- WhatsApp
- Feishu（飞书）
- QQ
- Slack
- Email
- Matrix
- WeCom（企业微信）
- Mochat

### 代码结构分析

Channels模块采用插件式架构，每个渠道都有独立的实现文件。以Telegram为例：

```python
# channels/telegram.py
class TelegramChannel:
    def __init__(self, config):
        self.token = config.get('token')
        self.allow_from = config.get('allowFrom', [])
        self.group_policy = config.get('groupPolicy', 'mention')
        
    async def connect(self):
        # 初始化Telegram bot客户端
        pass
        
    async def handle_message(self, message):
        # 处理接收到的消息
        if self._should_respond(message):
            await self._process_message(message)
            
    def _should_respond(self, message):
        # 根据allowFrom和groupPolicy决定是否响应
        pass
```

### 关键设计模式

1. **统一接口**：所有渠道都实现相同的接口方法
2. **配置驱动**：通过配置文件控制渠道行为
3. **安全优先**：默认deny-all策略，需要显式配置allowFrom

### 安全机制

nanobot在v0.1.4.post4版本后采用了更严格的安全策略：
- 空的`allowFrom`列表默认拒绝所有发送者
- 要允许所有发送者，必须显式设置`"allowFrom": ["*"]`
- 支持细粒度的群组策略控制

## Providers模块设计

### 多提供商支持

nanobot支持广泛的LLM提供商，包括：
- OpenRouter（推荐，访问所有模型）
- Anthropic（Claude直接）
- Azure OpenAI
- OpenAI（GPT直接）
- DeepSeek
- Groq（支持Whisper语音转录）
- Gemini
- MiniMax
- VolcEngine（火山引擎）
- Dashscope（Qwen）
- Moonshot/Kimi
- Zhipu GLM
- 本地模型（Ollama, vLLM）

### Provider Registry设计

nanobot使用Provider Registry作为单一事实源，这是其扩展性的关键：

```python
# providers/registry.py
PROVIDERS = [
    ProviderSpec(
        name="openrouter",
        keywords=("openrouter", "claude", "gpt"),
        env_key="OPENROUTER_API_KEY",
        display_name="OpenRouter",
        litellm_prefix="openrouter",
        skip_prefixes=("openrouter/",),
    ),
    # ... 其他提供商
]
```

### 添加新提供商的两步法

nanobot的设计使得添加新提供商变得极其简单：

**步骤1**：在`providers/registry.py`中添加ProviderSpec

```python
ProviderSpec(
    name="myprovider",
    keywords=("myprovider", "mymodel"),
    env_key="MYPROVIDER_API_KEY",
    display_name="My Provider",
    litellm_prefix="myprovider",
    skip_prefixes=("myprovider/",),
)
```

**步骤2**：在`config/schema.py`中添加字段

```python
class ProvidersConfig(BaseModel):
    myprovider: ProviderConfig = ProviderConfig()
```

这种设计避免了传统的if-elif链，使得代码更加简洁和可维护。

### 高级配置选项

ProviderSpec支持多种高级配置：

| 字段 | 描述 | 示例 |
|------|------|------|
| `litellm_prefix` | 自动为模型名添加前缀 | `"dashscope"` → `dashscope/qwen-max` |
| `skip_prefixes` | 避免重复添加前缀 | `("dashscope/", "openrouter/")` |
| `env_extras` | 额外的环境变量 | `(("ZHIPUAI_API_KEY", "{api_key}"),)` |
| `model_overrides` | 模型特定参数覆盖 | `(("kimi-k2.5", {"temperature": 1.0}),)` |
| `is_gateway` | 标记为网关提供商 | `True` |
| `detect_by_key_prefix` | 通过API密钥前缀检测 | `"sk-or-"` |

## Skills模块设计

### 技能系统架构

nanobot的Skills模块提供了功能扩展机制。技能可以是：
- 内置技能（github, weather, tmux等）
- 托管技能（通过ClawHub）
- 工作区技能（用户自定义）

### 技能加载机制

```python
# skills/loader.py
class SkillsLoader:
    def __init__(self, workspace_path):
        self.workspace_path = workspace_path
        self.bundled_skills = self._load_bundled_skills()
        self.managed_skills = self._load_managed_skills()
        self.workspace_skills = self._load_workspace_skills()
        
    def _load_bundled_skills(self):
        # 加载内置技能
        pass
        
    def get_skill(self, skill_name):
        # 按优先级查找技能：工作区 > 托管 > 内置
        pass
```

### 技能目录结构

每个技能都是一个包含SKILL.md的目录：

```
skills/
├── github/
│   └── SKILL.md
├── weather/
│   └── SKILL.md
└── my-custom-skill/
    └── SKILL.md
```

### ClowHub集成

nanobot集成了ClawHub技能注册表，可以自动搜索和安装公共技能：

```
Read https://clawhub.com/skills/github and install it
```

## 模块间协作

### 消息流

1. **Channels**接收外部消息
2. 消息通过**Bus**路由到**Agent**
3. **Agent**调用**Providers**获取LLM响应
4. LLM可能调用**Skills**执行工具操作
5. 结果通过**Channels**返回给用户

### 配置统一性

所有模块共享统一的配置系统：

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN"
    }
  },
  "providers": {
    "openrouter": {
      "apiKey": "sk-or-v1-xxx"
    }
  },
  "tools": {
    "mcpServers": {
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
      }
    }
  }
}
```

## 实际应用示例

### 多实例部署

nanobot支持多实例运行，每个实例可以有不同的配置：

```bash
# Telegram实例
nanobot gateway --config ~/.nanobot-telegram/config.json

# Discord实例  
nanobot gateway --config ~/.nanobot-discord/config.json

# Feishu实例
nanobot gateway --config ~/.nanobot-feishu/config.json --port 18792
```

### MCP集成

nanobot支持MCP（Model Context Protocol），可以连接外部工具服务器：

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

## 总结

nanobot的模块设计体现了其"超轻量级但功能完整"的核心理念：

1. **Channels模块**：提供多平台集成，安全优先
2. **Providers模块**：支持广泛的LLM提供商，扩展性极佳
3. **Skills模块**：灵活的技能系统，支持自定义扩展

这种模块化设计使得nanobot既保持了代码的简洁性（约4000行核心代码），又提供了企业级的功能完整性。开发者可以轻松地添加新的渠道、提供商或技能，而无需修改核心代码。

在下一篇文章中，我们将深入探讨nanobot的关键代码实现，包括代理循环、上下文构建、内存管理和子代理执行等核心功能。