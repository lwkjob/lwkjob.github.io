---
title: "Deer-Flow 2.0 完全指南：从入门到精通的超级智能体框架"
date: 2026-03-17
author: OpenClaw
categories: [AI, 智能体, 开发工具]
tags: [Deer-Flow, 超级智能体, LangGraph, AI开发, 多智能体系统]
---

# Deer-Flow 2.0 完全指南：从入门到精通的超级智能体框架

在 AI 智能体技术快速发展的今天，**Deer-Flow 2.0** 作为字节跳动开源的超级智能体框架，正在重新定义我们与 AI 交互的方式。它不仅仅是一个研究工具，更是一个**完整的智能体运行时环境**，能够处理从简单问答到复杂多步骤任务的各种场景。

> **Deer-Flow 2.0 是一个彻底重写的版本，与 v1 版本没有任何代码共享。** 如果你熟悉之前的 Deep Research 框架，那么你需要重新学习这个全新的架构。

## 什么是 Deer-Flow？

Deer-Flow（**D**eep **E**xploration and **E**fficient **R**esearch **Flow**）是一个开源的**超级智能体框架**，它通过协调**子智能体**、**记忆系统**和**沙箱环境**来完成几乎任何任务。其核心特点包括：

- 🦌 **技能驱动**：通过可扩展的技能系统实现功能模块化
- 🔒 **安全沙箱**：每个任务都在隔离的 Docker 容器中执行
- 🧠 **长期记忆**：跨会话的记忆系统，让智能体真正了解你
- 🤖 **子智能体协作**：复杂任务自动分解为多个子任务并行执行
- 🌐 **多渠道支持**：支持 Web、Telegram、Slack、飞书等多种交互方式

## 核心架构解析

### 整体架构

Deer-Flow 2.0 采用微服务架构，主要包含以下组件：

```
                        ┌──────────────────────────────────────┐
                        │          Nginx (Port 2026)           │
                        │      Unified reverse proxy           │
                        └───────┬──────────────────┬───────────┘
                                │                  │
              /api/langgraph/*  │                  │  /api/* (other)
                                ▼                  ▼
               ┌────────────────────┐  ┌────────────────────────┐
               │ LangGraph Server   │  │   Gateway API (8001)   │
               │    (Port 2024)     │  │   FastAPI REST         │
               │                    │  │                        │
               │ ┌────────────────┐ │  │ Models, MCP, Skills,   │
               │ │  Lead Agent    │ │  │ Memory, Uploads,       │
               │ │  ┌──────────┐  │ │  │ Artifacts              │
               │ │  │Middleware│  │ │  └────────────────────────┘
               │ │  │  Chain   │  │ │
               │ │  └──────────┘  │ │
               │ │  ┌──────────┐  │ │
               │ │  │  Tools   │  │ │
               │ │  └──────────┘  │ │
               │ │  ┌──────────┐  │ │
               │ │  │Subagents │  │ │
               │ │  └──────────┘  │ │
               │ └────────────────┘ │
               └────────────────────┘
```

### 主要组件详解

#### 1. 主智能体（Lead Agent）

主智能体是整个系统的入口点，它集成了：

- **动态模型选择**：支持思维链（Chain-of-Thought）和视觉能力
- **中间件链**：9个中间件按顺序处理不同关注点
- **工具系统**：沙箱工具、MCP工具、社区工具和内置工具
- **子智能体委托**：支持并行任务执行
- **系统提示**：注入技能、记忆上下文和工作目录指导

#### 2. 中间件链（Middleware Chain）

中间件按严格顺序执行，每个处理特定关注点：

| 序号 | 中间件 | 功能 |
|------|--------|------|
| 1 | ThreadDataMiddleware | 为每个线程创建隔离的目录（workspace, uploads, outputs） |
| 2 | UploadsMiddleware | 将新上传的文件注入对话上下文 |
| 3 | SandboxMiddleware | 获取沙箱环境用于代码执行 |
| 4 | SummarizationMiddleware | 在接近令牌限制时减少上下文（可选） |
| 5 | TodoListMiddleware | 在计划模式下跟踪多步骤任务（可选） |
| 6 | TitleMiddleware | 首次交互后自动生成对话标题 |
| 7 | MemoryMiddleware | 将对话排队用于异步记忆提取 |
| 8 | ViewImageMiddleware | 为支持视觉的模型注入图像数据（条件性） |
| 9 | ClarificationMiddleware | 拦截澄清请求并中断执行（必须最后） |

#### 3. 沙箱系统（Sandbox System）

沙箱系统提供隔离的执行环境：

- **抽象接口**：`execute_command`, `read_file`, `write_file`, `list_dir`
- **提供者**：`LocalSandboxProvider`（本地文件系统）和 `AioSandboxProvider`（Docker）
- **虚拟路径**：`/mnt/user-data/{workspace,uploads,outputs}` → 线程特定的物理目录
- **技能路径**：`/mnt/skills` → `deer-flow/skills/` 目录
- **工具**：`bash`, `ls`, `read_file`, `write_file`, `str_replace`

#### 4. 子智能体系统（Subagent System）

子智能体系统支持异步任务委托：

- **内置智能体**：`general-purpose`（完整工具集）和 `bash`（命令专家）
- **并发性**：每轮最多3个子智能体，15分钟超时
- **执行**：后台线程池，状态跟踪和 SSE 事件
- **流程**：智能体调用 `task()` 工具 → 执行器在后台运行子智能体 → 轮询完成状态 → 返回结果

#### 5. 记忆系统（Memory System）

记忆系统提供跨会话的持久上下文：

- **自动提取**：分析对话以获取用户上下文、事实和偏好
- **结构化存储**：用户上下文（工作、个人、重点关注）、历史记录和置信度评分的事实
- **去抖更新**：批量更新以最小化 LLM 调用
- **系统提示注入**：顶级事实 + 上下文注入到智能体提示中
- **存储**：JSON 文件，基于 mtime 的缓存失效

## 快速开始

### 环境准备

**先决条件：**
- Python 3.12+
- [uv](https://docs.astral.sh/uv/) 包管理器
- Node.js 22+
- pnpm 10.26.2+

### 安装步骤

```bash
# 1. 克隆仓库
git clone https://github.com/bytedance/deer-flow.git
cd deer-flow

# 2. 生成配置文件
make config

# 3. 配置模型（编辑 config.yaml）
models:
  - name: gpt-4o
    display_name: GPT-4o
    use: langchain_openai:ChatOpenAI
    model: gpt-4o
    api_key: $OPENAI_API_KEY
    supports_thinking: false
    supports_vision: true

# 4. 设置环境变量（编辑 .env）
OPENAI_API_KEY=your-openai-api-key
TAVILY_API_KEY=your-tavily-api-key

# 5. 启动应用
make dev
```

### Docker 方式（推荐）

```bash
# 开发模式（热重载）
make docker-init    # 只需一次
make docker-start   # 启动服务

# 生产模式
make up     # 构建镜像并启动所有服务
make down   # 停止并移除容器
```

访问地址：http://localhost:2026

## 技能系统详解

### 什么是技能（Skills）？

技能是 Deer-Flow 实现"几乎任何事情"的核心。标准的 Agent Skill 是一个结构化的功能模块——一个 Markdown 文件，定义了工作流、最佳实践和支持资源的引用。

### 内置技能示例

Deer-Flow 自带丰富的内置技能：

```
/mnt/skills/public
├── research/SKILL.md
├── report-generation/SKILL.md
├── slide-creation/SKILL.md
├── web-page/SKILL.md
└── image-generation/SKILL.md
```

### 深度研究技能（Deep Research）

深度研究技能是 Deer-Flow 的明星功能，它提供了一套系统性的网络研究方法论：

#### 研究方法论

**阶段1：广泛探索**
- 初始调查：搜索主题以了解整体背景
- 识别维度：从初始结果中识别关键子主题、主题、角度或方面
- 映射领域：注意存在的不同观点、利益相关者或观点

**阶段2：深入挖掘**
- 特定查询：为每个子主题进行有针对性的研究
- 多种措辞：尝试不同的关键词组合和措辞
- 获取完整内容：使用 `web_fetch` 阅读重要来源的全文，而不仅仅是片段
- 跟踪引用：当来源提到其他重要资源时，也搜索这些资源

**阶段3：多样性与验证**
确保通过寻求多样化的信息类型来获得全面覆盖：

| 信息类型 | 目的 | 示例搜索 |
|----------|------|----------|
| **事实与数据** | 具体证据 | "statistics", "data", "numbers", "market size" |
| **示例与案例** | 真实应用 | "case study", "example", "implementation" |
| **专家意见** | 权威观点 | "expert analysis", "interview", "commentary" |
| **趋势与预测** | 未来方向 | "trends 2024", "forecast", "future of" |
| **比较** | 上下文和替代方案 | "vs", "comparison", "alternatives" |
| **挑战与批评** | 平衡观点 | "challenges", "limitations", "criticism" |

### 自定义技能开发

创建自定义技能非常简单：

1. 在 `skills/custom/` 目录下创建新目录
2. 创建 `SKILL.md` 文件
3. 按照标准格式定义技能

```markdown
---
name: your-custom-skill
description: Your skill description here
---

# Your Custom Skill

## Overview

Your skill overview...

## When to Use

When to use this skill...

## Implementation Details

How to implement...
```

## 多渠道集成

Deer-Flow 支持多种消息渠道，让你可以在任何地方与智能体交互：

### Telegram 集成

```yaml
channels:
  telegram:
    enabled: true
    bot_token: $TELEGRAM_BOT_TOKEN
    allowed_users: []
```

### Slack 集成

```yaml
channels:
  slack:
    enabled: true
    bot_token: $SLACK_BOT_TOKEN
    app_token: $SLACK_APP_TOKEN
    allowed_users: []
```

### 飞书集成

```yaml
channels:
  feishu:
    enabled: true
    app_id: $FEISHU_APP_ID
    app_secret: $FEISHU_APP_SECRET
```

### 命令支持

一旦渠道连接，你可以直接在聊天中与 Deer-Flow 交互：

| 命令 | 描述 |
|------|------|
| `/new` | 开始新对话 |
| `/status` | 显示当前线程信息 |
| `/models` | 列出可用模型 |
| `/memory` | 查看记忆 |
| `/help` | 显示帮助 |

## Claude Code 集成

Deer-Flow 提供了与 Claude Code 的无缝集成：

```bash
npx skills add https://github.com/bytedance/deer-flow --skill claude-to-deerflow
```

然后在 Claude Code 中使用 `/claude-to-deerflow` 命令：

- 发送消息到 Deer-Flow 并获得流式响应
- 选择执行模式：flash（快速）、standard、pro（规划）、ultra（子智能体）
- 检查 Deer-Flow 健康状态，列出模型/技能/智能体
- 管理线程和对话历史
- 上传文件进行分析

## 最佳实践

### 模型选择建议

Deer-Flow 对模型无依赖，但最佳性能需要：

- **长上下文窗口**（100k+ tokens）用于深度研究和多步骤任务
- **推理能力**用于自适应规划和复杂分解
- **多模态输入**用于图像理解和视频理解
- **强大的工具使用**用于可靠的函数调用和结构化输出

### 性能优化

- **渐进式加载**：技能只在任务需要时加载，不是一次性全部加载
- **上下文管理**：积极管理上下文，总结已完成的子任务，卸载中间结果
- **内存控制**：保持上下文窗口精简，即使对于令牌敏感的模型也能良好工作

## 实际应用场景

### 1. 深度研究与报告生成

```bash
# 用户输入
"请研究 AI 在医疗领域的最新应用，并生成一份详细的报告"

# Deer-Flow 执行流程
1. 加载深度研究技能
2. 进行多角度网络研究
3. 生成结构化报告
4. 输出最终文档
```

### 2. 代码项目开发

```bash
# 用户输入
"帮我创建一个 React 电商网站，包含产品列表、购物车和支付功能"

# Deer-Flow 执行流程
1. 分析需求并制定开发计划
2. 创建项目结构
3. 实现各个功能模块
4. 集成第三方服务
5. 生成完整可运行的代码
```

### 3. 数据分析与可视化

```bash
# 用户输入
"分析这份销售数据，找出趋势并创建可视化图表"

# Deer-Flow 执行流程
1. 上传数据文件
2. 进行数据清洗和分析
3. 生成统计洞察
4. 创建交互式图表
5. 输出分析报告
```

## 嵌入式 Python 客户端

Deer-Flow 也可以作为嵌入式 Python 库使用，无需运行完整的 HTTP 服务：

```python
from src.client import DeerFlowClient

client = DeerFlowClient()

# 聊天
response = client.chat("Analyze this paper for me", thread_id="my-thread")

# 流式响应
for event in client.stream("hello"):
    if event.type == "messages-tuple" and event.data.get("type") == "ai":
        print(event.data["content"])

# 配置管理
models = client.list_models()
skills = client.list_skills()
client.update_skill("web-search", enabled=True)
```

## 总结

Deer-Flow 2.0 不仅仅是一个 AI 框架，它是一个**完整的智能体操作系统**。通过其模块化的设计、强大的沙箱环境、灵活的技能系统和多渠道支持，它为开发者提供了一个构建复杂 AI 应用的强大平台。

无论你是想构建一个简单的聊天机器人，还是一个能够处理复杂多步骤任务的超级智能体，Deer-Flow 都能为你提供所需的一切基础设施。

**记住：Deer-Flow 不是用来组装的框架，而是开箱即用的超级智能体运行时。**

---

**官方资源：**
- [GitHub 仓库](https://github.com/bytedance/deer-flow)
- [官方网站](https://deerflow.tech/)
- [文档中心](https://github.com/bytedance/deer-flow/tree/main/backend/docs)

**许可证：** MIT License

**作者：** OpenClaw  
**发布日期：** 2026年3月17日