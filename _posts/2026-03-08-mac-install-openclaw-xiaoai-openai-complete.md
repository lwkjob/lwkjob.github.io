---
layout: post
title: "Mac电脑安装OpenClaw并使用小爱链接OpenAI"
date: 2026-03-08 20:30:00 +0800
categories: [AI, OpenClaw, 教程]
tags: [macOS, OpenClaw, 小爱同学, OpenAI, AI助手]
---

## 前言

OpenClaw是一个强大的开源AI助手框架，可以让你在本地部署智能助手，并通过各种渠道（如QQ、微信、小爱同学等）进行交互。本文将详细介绍如何在Mac电脑上安装OpenClaw，并配置小爱同学来链接OpenAI服务。

## 系统要求

- **操作系统**: macOS 10.15 (Catalina) 或更高版本
- **Node.js**: 版本 ≥ 22.0.0
- **网络环境**: 稳定的网络连接（建议配置国内镜像加速）

## 安装步骤

### 1. 安装Node.js

推荐使用Homebrew安装：

```bash
# 安装Homebrew（如果未安装）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 使用Homebrew安装Node.js
brew install node@22

# 验证安装
node --version
npm --version
```

### 2. 配置国内镜像（可选但推荐）

```bash
# 设置npm国内镜像源
npm config set registry https://registry.npmmirror.com
```

### 3. 安装OpenClaw

有两种安装方式：

**方式一：官方一键脚本（推荐新手）**
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
```

**方式二：npm安装**
```bash
npm install -g openclaw@latest
```

如果遇到权限问题，可以使用：
```bash
sudo npm install -g openclaw@latest
```

### 4. 初始化配置

安装完成后，运行配置向导：
```bash
openclaw onboard
```

按照提示完成以下配置：
- 选择快速开始模式（QuickStart）
- 配置AI模型API Key（可以使用阿里云百炼、Kimi、DeepSeek等）
- 渠道配置（如飞书、QQ等，可暂跳过）
- 技能配置（Skills，可暂跳过）

### 5. 启动服务

配置完成后，启动OpenClaw网关服务：
```bash
openclaw gateway
```

成功启动后，终端会显示Web UI访问地址（如 `http://127.0.0.1:18789`），保持终端运行即可通过浏览器访问。

## 配置小爱同学链接

### 1. 获取API Key

首先需要获取支持的AI模型API Key，推荐使用：
- 阿里云百炼API Key
- Kimi API Key  
- DeepSeek API Key

### 2. 配置小爱同学渠道

在OpenClaw Web UI中配置小爱同学渠道，或者通过命令行配置：

```bash
# 配置小爱同学渠道（具体参数根据实际情况调整）
openclaw channels configure xiaomi
```

### 3. 测试连接

在小爱同学中发送测试消息，如"你好"，确认AI响应正常即配置成功。

## 常见问题

### 1. 安装缓慢
- 配置国内镜像源
- 确保网络连接稳定
- 使用代理（如有需要）

### 2. 权限问题
- 使用 `sudo` 命令
- 或通过 `nvm` 管理Node.js版本

### 3. 服务启动失败
- 检查端口是否被占用
- 确认Node.js版本兼容性
- 查看日志文件排查具体错误

## 总结

通过以上步骤，你就可以在Mac电脑上成功安装OpenClaw，并配置小爱同学来使用AI服务。OpenClaw的强大之处在于其模块化设计，支持多种AI模型和通信渠道，可以根据个人需求进行灵活配置。

## 参考资料

- OpenClaw官方文档
- 阿里云百炼API文档
- Node.js官方安装指南

---
*本文基于实际安装经验编写，如有疑问欢迎交流讨论。*