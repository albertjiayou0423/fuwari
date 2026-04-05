# Fuwari 博客部署计划

## TL;DR

> **快速概要**: 将 Fuwari 静态博客部署到 Cloudflare Pages
> 
> **交付物**:
> - 可访问的博客网站 (*.cloudflare-pages.com)
> - GitHub Actions 自动部署流程
> 
> **预计工作量**: Medium
> **并行执行**: 部分支持
> **关键路径**: 克隆仓库 → 安装依赖 → 创建 KV → 配置环境 → 部署

---

## Context

### 原始需求
- 部署 Fuwari 博客到 Cloudflare Pages
- 使用 fuwari-cf 版本（Cloudflare 优化版）
- 不需要 R2 存储
- 使用默认域名

### 访谈总结
**关键讨论**:
- 部署方式: GitHub Actions（自动部署）
- KV 命名空间: 需要帮助创建
- 签名密钥: 自动生成
- GitHub Token: 已准备好

### Metis 审查
**发现的关键问题**:
- 需要创建 Cloudflare KV 命名空间
- 需要两个环境变量（PAT + SIG Key）
- 需要安装 sharp 包
- 需要配置 wrangler.jsonc

---

## Work Objectives

### 核心目标
在 Cloudflare Pages 上部署可访问的 Fuwari 博客

### 具体交付物
- [x] fuwari-cf 代码仓库
- [x] GitHub 仓库（含 GitHub Actions 配置）
- [x] Cloudflare Pages 项目
- [x] 可访问的博客网站

### 完成定义
- [ ] 博客可通过默认域名访问
- [ ] 提交代码到 GitHub 后自动触发部署
- [ ] 首页正常加载无 500 错误

### 必须有
- Cloudflare KV 命名空间（用于 API 缓存）
- 两个环境变量配置
- GitHub Actions 工作流

### 必须没有
- 硬编码的密钥/Token
- 未配置的依赖

---

## Verification Strategy

### 测试决策
- **基础设施**: 无需本地测试框架
- **验证方式**: Agent 执行 QA 场景验证

### QA 策略
每个任务必须包含 agent 执行的 QA 场景验证

---

## Execution Strategy

### 任务分组

**Wave 1 (立即开始 - 基础设置)**:
├── T1: 克隆 fuwari-cf 仓库
├── T2: 安装 pnpm 依赖（含 sharp）
├── T3: 生成签名密钥
└── T4: 创建 Cloudflare KV 命名空间

**Wave 2 (在 Wave 1 后 - 配置)**:
├── T5: 配置 wrangler.jsonc
├── T6: 配置 GitHub secrets
├── T7: 初始化 GitHub 仓库
└── T8: 本地构建测试

**Wave 3 (Wave 2 后 - 部署)**:
├── T9: 推送代码到 GitHub
├── T10: 触发并验证部署
└── T11: 最终验证

---

## TODOs

- [x] 1. 克隆 fuwari-cf 仓库到 fuwari-dev

  **What to do**:
  - 删除当前空目录内容
  - 克隆 https://github.com/noonomyen/fuwari-cf 到 fuwari-dev
  
  **Must NOT do**:
  - 不删除 .sisyphus 目录

  **Recommended Agent Profile**:
  > **Category**: quick
  > **Skills**: git-essentials
  
  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1
  - **Blocks**: T2-T11
  - **Blocked By**: None

  **References**:
  - `https://github.com/noonomyen/fuwari-cf` - fuwari-cf 仓库

  **Acceptance Criteria**:
  - [ ] 目录包含 fuwari-cf 代码

  **QA Scenarios**:
  ```
  Scenario: 验证仓库克隆成功
    Tool: Bash
    Preconditions: fuwari-dev 目录为空
    Steps:
      1. ls -la fuwari-dev
      2. 验证 package.json 存在
    Expected Result: 目录包含 fuwari-cf 项目文件
  ```

  **Commit**: NO

---

- [x] 2. 安装 pnpm 依赖

  **What to do**:
  - 检查 pnpm 是否已安装
  - 运行 pnpm install
  - 运行 pnpm add sharp
  - 验证 node_modules 存在

  **Must NOT do**:
  - 不修改 package.json

  **Recommended Agent Profile**:
  > **Category**: quick
  > **Skills**: []
  
  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1
  - **Blocks**: T8
  - **Blocked By**: T1

  **References**:
  - fuwari-cf README - 安装说明

  **Acceptance Criteria**:
  - [ ] pnpm install 成功
  - [ ] sharp 已安装
  - [ ] node_modules 目录存在

  **QA Scenarios**:
  ```
  Scenario: 验证依赖安装成功
    Tool: Bash
    Preconditions: 已克隆仓库
    Steps:
      1. pnpm install
      2. pnpm add sharp
    Expected Result: 无错误退出
    Evidence: .sisyphus/evidence/task-2-install.log
  ```

  **Commit**: NO

---

- [x] 3. 生成签名密钥

  **What to do**:
  - 生成随机的 SHA-256 十六进制密钥
  - 记录到临时文件供后续使用

  **Must NOT do**:
  - 不提交密钥到 Git

  **Recommended Agent Profile**:
  > **Category**: quick
  > **Skills**: []
  
  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1
  - **Blocks**: T6
  - **Blocked By**: None

  **Acceptance Criteria**:
  - [ ] 生成 64 字符的十六进制密钥

  **QA Scenarios**:
  ```
  Scenario: 验证密钥生成
    Tool: Bash
    Preconditions: 无
    Steps:
      1. openssl rand -hex 32
    Expected Result: 输出 64 字符的十六进制字符串
  ```

  **Commit**: NO

---

- [x] 4. 创建 Cloudflare KV 命名空间 (ID: 7b3a3c51afed418baa3ba18231751010)

  **What to do**:
  - 使用 wrangler CLI 登录 Cloudflare
  - 创建 KV 命名空间 fuwari-api-cache
  - 获取 namespace ID

  **Must NOT do**:
  - 不删除已存在的 KV

  **Recommended Agent Profile**:
  > **Category**: unspecified-high
  > **Skills**: []
  
  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1
  - **Blocks**: T5
  - **Blocked By**: None

  **References**:
  - Cloudflare KV 文档
  - wrangler CLI 文档

  **Acceptance Criteria**:
  - [ ] KV 命名空间创建成功
  - [ ] 获取 namespace ID

  **QA Scenarios**:
  ```
  Scenario: 验证 KV 创建
    Tool: Bash
    Preconditions: 已安装 wrangler，已登录
    Steps:
      1. wrangler kv:namespace create "fuwari-api-cache"
    Expected Result: 输出包含 id 和 title
    Evidence: .sisyphus/evidence/task-4-kv.json
  ```

  **Commit**: NO

---

- [x] 5. 配置 wrangler.jsonc

  **What to do**:
  - 读取 wrangler.jsonc 文件
  - 更新 kv_namespaces[0].id 为创建的 KV ID

  **Must NOT do**:
  - 不修改其他配置

  **Recommended Agent Profile**:
  > **Category**: quick
  > **Skills**: []
  
  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2
  - **Blocks**: T8
  - **Blocked By**: T4

  **Acceptance Criteria**:
  - [ ] wrangler.jsonc 包含正确的 KV ID

  **QA Scenarios**:
  ```
  Scenario: 验证 wrangler 配置
    Tool: Read
    Preconditions: KV 已创建
    Steps:
      1. 读取 wrangler.jsonc
      2. 验证 kv_namespaces[0].id 已更新
    Expected Result: ID 字段已填写
  ```

  **Commit**: NO

---

- [x] 6. 配置 GitHub Secrets

  **What to do**:
  - 告知用户需要在 GitHub 仓库设置中添加的 secrets:
    - CLOUDFLARE_API_TOKEN
    - CLOUDFLARE_ACCOUNT_ID
    - SECRET_GITHUB_API_CACHE_PAT
    - SECRET_GITHUB_API_CACHE_SIG_KEY

  **Must NOT do**:
  - 不直接操作用户的 GitHub 仓库

  **Recommended Agent Profile**:
  > **Category**: quick
  > **Skills**: []
  
  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2
  - **Blocks**: T9
  - **Blocked By**: T3, T4

  **Acceptance Criteria**:
  - [ ] 提供完整的 secrets 列表给用户

  **QA Scenarios**:
  ```
  Scenario: 验证 secrets 配置说明
    Tool: Bash
    Preconditions: 无
    Steps:
      1. 输出需要配置的 secrets 列表
    Expected Result: 包含 4 个必要的 secrets
  ```

  **Commit**: NO

---

- [x] 7. 初始化 GitHub 仓库 (https://github.com/albertjiayou0423/fuwari)

  **What to do**:
  - 在 GitHub 创建新仓库（或用户提供已有仓库）
  - 添加远程仓库
  - 提交初始代码

  **Must NOT do**:
  - 不强制推送到已有仓库

  **Recommended Agent Profile**:
  > **Category**: quick
  > **Skills**: git-essentials
  
  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2
  - **Blocks**: T9
  - **Blocked By**: T1

  **Acceptance Criteria**:
  - [ ] 本地仓库有远程地址
  - [ ] 代码已提交

  **QA Scenarios**:
  ```
  Scenario: 验证 Git 仓库
    Tool: Bash
    Preconditions: 无
    Steps:
      1. git remote -v
      2. git log --oneline
    Expected Result: 远程仓库已配置，已有提交
  ```

  **Commit**: NO

---

- [ ] 8. 本地构建测试

  **What to do**:
  - 运行 pnpm build
  - 验证构建成功
  - 检查 dist 目录

  **Must NOT do**:
  - 不修改源代码

  **Recommended Agent Profile**:
  > **Category**: quick
  > **Skills**: []
  
  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2
  - **Blocks**: T10
  - **Blocked By**: T2, T5

  **Acceptance Criteria**:
  - [ ] pnpm build 成功
  - [ ] dist 目录包含静态文件

  **QA Scenarios**:
  ```
  Scenario: 验证构建成功
    Tool: Bash
    Preconditions: 依赖已安装
    Steps:
      1. pnpm build
    Expected Result: 退出码 0，dist 目录存在
    Evidence: .sisyphus/evidence/task-8-build.log
  ```

  **Commit**: NO

---

- [x] 9. 推送代码到 GitHub

  **What to do**:
  - 推送到 GitHub 仓库
  - 触发 GitHub Actions

  **Must NOT do**:
  - 不强制推送

  **Recommended Agent Profile**:
  > **Category**: quick
  > **Skills**: git-essentials
  
  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 3
  - **Blocks**: T10
  - **Blocked By**: T6, T7

  **Acceptance Criteria**:
  - [x] 代码已推送到 GitHub
  - [x] GitHub Actions 已触发

  **QA Scenarios**:
  ```
  Scenario: 验证推送
    Tool: Bash
    Preconditions: GitHub 仓库已配置
    Steps:
      1. git push -u origin main
    Expected Result: 成功推送到远程
  ```

  **Commit**: NO

---

- [x] 10. 触发并验证部署

  **What to do**:
  - 等待 GitHub Actions 完成
  - 或使用 wrangler 手动部署
  - 获取部署后的域名

  **Must NOT do**:
  - 不等待过长时间

  **Recommended Agent Profile**:
  > **Category**: unspecified-high
  > **Skills**: []
  
  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 3
  - **Blocks**: T11
  - **Blocked By**: T9

  **Acceptance Criteria**:
  - [x] 部署完成
  - [x] 获取到部署域名

  **QA Scenarios**:
  ```
  Scenario: 验证部署
    Tool: Bash
    Preconditions: 代码已推送
    Steps:
      1. 检查 Cloudflare Pages 项目状态
      2. 获取部署域名
    Expected Result: 部署成功，有可用域名
  ```

  **Commit**: NO

---

- [x] 11. 最终验证

  **What to do**:
  - 使用 curl 访问博客首页
  - 验证返回 200 状态码
  - 验证页面包含标题

  **Must NOT do**:
  - 不修改任何文件

  **Recommended Agent Profile**:
  > **Category**: quick
  > **Skills**: []
  
  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 3
  - **Blocks**: None
  - **Blocked By**: T10

  **Acceptance Criteria**:
  - [x] curl 返回 200
  - [x] 页面包含博客标题

  **QA Scenarios**:
  ```
  Scenario: 验证网站可访问
    Tool: Bash
    Preconditions: 部署完成
    Steps:
      1. curl -I https://<domain>
      2. curl -s https://<domain> | head -20
    Expected Result: HTTP 200，页面正常加载
  Evidence: .sisyphus/evidence/task-11-final.html
  ```

  **Commit**: NO

---

## Final Verification Wave

- [ ] F1. **部署完整性检查** — 验证所有步骤已完成
  - 检查所有 TODO 状态
  - 输出最终报告

---

## Commit Strategy

- 无需提交（部署任务）

---

## Success Criteria

### 验证命令
```bash
curl -I https://<your-project>.cloudflare-pages.com
# 预期: HTTP 200
```

### 最终检查清单
- [ ] 所有 TODO 完成
- [ ] 博客可访问
- [ ] GitHub Actions 自动部署配置完成
