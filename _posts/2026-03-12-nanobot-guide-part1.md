---
layout: post
title: "nanobot 入门指南：超轻量级 AI 助手的快速上手"
date: 2026-03-12 03:00:00 +0800
categories: [AI, nanobot, tutorial]
tags: [nanobot, AI assistant, lightweight, tutorial]
---

## 什么是 nanobot？

nanobot 是一个超轻量级的个人 AI 助手，灵感来源于 OpenClaw，但代码量减少了 99%。它只包含约 4,000 行核心代理代码，却提供了完整的 AI 助手功能。

**核心特点：**
- 🪶 **超轻量级**：仅 ~4,000 行核心代码
- 🔬 **研究友好**：代码简洁易读，便于理解和修改
- ⚡️ **闪电快速**：启动快，资源占用少，迭代迅速
- 💎 **易于使用**：一键部署，快速上手

## 安装 nanobot

nanobot 提供了多种安装方式：

### 1. 从源码安装（推荐用于开发）
```bash
git clone https://github.com/HKUDS/nanobot.git
cd nanobot
pip install -e .
```

### 2. 使用 uv 安装（稳定快速）
```bash
uv tool install nanobot-ai
```

### 3. 从 PyPI 安装（稳定版本）
```bash
pip install nanobot-ai
```

## 快速开始

### 步骤 1：初始化配置
```bash
nanobot onboard
```
这个命令会创建默认的配置文件和工作区目录。

### 步骤 2：配置 API 密钥
编辑 `~/.nanobot/config.json` 文件，添加您的 LLM 提供商 API 密钥：

```json
{
  "providers": {
    "openrouter": {
      "apiKey": "sk-or-v1-xxx"
    }
  }
}
```

### 步骤 3：设置模型
在同一配置文件中设置您想要使用的模型：

```json
{
  "agents": {
    "defaults": {
      "model": "anthropic/claude-opus-4-5",
      "provider": "openrouter"
    }
  }
}
```

### 步骤 4：开始聊天
```bash
nanobot agent
```

现在您已经拥有了一个工作的 AI 助手！

## 基本命令参考

| 命令 | 描述 |
|------|------|
| `nanobot onboard` | 初始化配置和工作区 |
| `nanobot agent -m "..."` | 与代理聊天 |
| `nanobot agent` | 交互式聊天模式 |
| `nanobot gateway` | 启动网关服务 |
| `nanobot status` | 显示状态信息 |
| `nanobot channels login` | 连接 WhatsApp（扫描二维码） |

## 配置文件结构

nanobot 的配置文件 `~/.nanobot/config.json` 包含以下主要部分：

1. **providers**: LLM 提供商配置
2. **agents**: 代理默认设置
3. **channels**: 聊天渠道配置
4. **tools**: 工具配置（包括 MCP 服务器）

## 支持的聊天平台

nanobot 支持多种聊天平台：
- **Telegram**（推荐）
- **Discord**
- **WhatsApp**
- **Feishu**（飞书）
- **QQ**
- **Slack**
- **Email**
- **Matrix**
- **DingTalk**（钉钉）
- **Wecom**（企业微信）

## 下一步

在下一篇中，我们将深入探讨 nanobot 的项目结构和核心模块设计，了解它是如何用如此少的代码实现完整功能的。

---

*提示：要获得最佳体验，建议使用 OpenRouter 作为 LLM 提供商，它可以访问所有主流模型。*