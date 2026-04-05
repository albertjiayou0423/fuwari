---
title: 使用 Cloudflare Pages 部署 Fuwari 静态博客
published: 2026-04-05
description: 记录如何使用 Cloudflare Pages + GitHub Actions 部署 Fuwari 静态博客
tags: [Cloudflare, Fuwari, Astro, 部署]
category: 技术
draft: false
lang: zh-CN
---

## 前言

最近成功将 Fuwari 博客部署到 Cloudflare Pages 上，整个过程还算顺利，特此记录一下部署过程，希望能帮助到想要部署博客的朋友。

## 什么是 Fuwari？

Fuwari 是一个基于 Astro 构建的静态博客模板，具有以下特点：

- 🌙 支持明暗主题切换
- 📱 响应式设计
- 🎨 可自定义主题颜色
- ⚡ 性能出色
- 🔧 支持 Cloudflare Pages

## 部署步骤

### 1. 准备工作

在开始之前，你需要准备以下内容：

- **GitHub 账号** - 用于托管代码和 CI/CD
- **Cloudflare 账号** - 用于托管博客
- **GitHub Personal Access Token (PAT)** - 用于 GitHub API 访问

### 2. 克隆项目

首先，克隆 fuwari-cf 仓库（专为 Cloudflare 优化的版本）：

```bash
git clone https://github.com/noonomyen/fuwari-cf your-blog
cd your-blog
```

### 3. 安装依赖

```bash
pnpm install
pnpm add sharp
```

### 4. 创建 Cloudflare KV 命名空间

Fuwari 使用 Cloudflare KV 来缓存 GitHub API 请求，提高访问速度。

```bash
wrangler kv:namespace create "fuwari-api-cache"
```

记下返回的 KV ID，后续需要用到。

### 5. 配置 wrangler.jsonc

编辑 `wrangler.jsonc`，将 KV ID 填入：

```json
{
  "kv_namespaces": [
    {
      "binding": "KV_GITHUB_API_CACHE",
      "id": "你的KV_ID"
    }
  ]
}
```

### 6. 配置环境变量

本地运行时需要创建 `.dev.vars` 文件：

```env
SECRET_GITHUB_API_CACHE_PAT=你的GitHub_PAT
SECRET_GITHUB_API_CACHE_SIG_KEY=你的签名密钥
```

签名密钥可以用以下命令生成：

```bash
openssl rand -hex 32
```

### 7. 配置 GitHub Secrets

在 GitHub 仓库的 Settings → Secrets and variables → Actions 中添加以下 Secrets：

| Secret 名称 | 说明 |
|-------------|------|
| CLOUDFLARE_API_TOKEN | Cloudflare API Token |
| CLOUDFLARE_ACCOUNT_ID | Cloudflare 账户 ID |
| SECRET_GITHUB_API_CACHE_PAT | GitHub PAT |
| SECRET_GITHUB_API_CACHE_SIG_KEY | 签名密钥 |

### 8. 修改配置文件

根据需要修改 `src/config.ts`：

- **title**: 网站标题
- **subtitle**: 副标题
- **lang**: 语言 (zh_CN)
- **themeColor.hue**: 主题色 (120 代表绿色系)
- **profileConfig.name**: 作者名称
- **profileConfig.bio**: 作者简介

### 9. 配置 GitHub Actions

确保 `.github/workflows/build.yml` 包含 Cloudflare Pages 部署步骤。

### 10. 部署

推送到 GitHub 后，GitHub Actions 会自动构建并部署到 Cloudflare Pages。

## 主题颜色

我把主题色设置为了 **120 度**（绿色系），这就是传说中的 120 风格 🎨

## 总结

整个部署过程比较顺畅，Cloudflare Pages 的免费额度也足够个人博客使用。Fuwari 本身是一个非常优秀的博客模板，强烈推荐！

如果你有任何问题，欢迎在评论区留言讨论。