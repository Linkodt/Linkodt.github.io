---
title: 介绍 updateWhat 项目及未来发展规划
date: 2026-04-17 10:30:00
tags: [前端, AI, 自动化, 开发工具, OpenClaw, MCP]
categories: 技术实践
description: 本文介绍了 updateWhat 项目 - 一个帮助开发者生成 GitHub 仓库版本升级指南的 AI 工具，并分享了未来的发展规划，包括创建 OpenClaw skill 和 MCP 服务。
---

# 介绍 updateWhat 项目及未来发展规划

## 引言

作为开发者，我们经常会遇到项目依赖升级的问题。当一个开源库发布新版本时，我们需要了解：有哪些破坏性变更？API 有什么变化？如何安全地升级？手动收集这些信息不仅耗时，而且容易遗漏重要细节。

今天，我要向大家介绍我们的 **updateWhat** 项目 - 一个利用 AI 智能生成版本升级指南的工具，以及我们对这个项目未来发展的规划。

## 什么是 updateWhat？

**updateWhat** 是一个帮助开发者生成 GitHub 仓库两个版本之间升级指南的 MVP Web 应用。它通过分析 GitHub releases 和 commit 信息，使用 AI 生成结构化的升级指南，让版本升级变得更加简单和安全。

### 核心功能特性

- 📊 **自动收集信息**：从 GitHub API 获取 releases 和 commits 对比信息
- 🎯 **智能筛选**：自动提取重要提交（破坏性变更、新功能、重构等）
- 🤖 **AI 总结**：使用 OpenAI 兼容 API 生成结构化升级指南
- 📝 **Markdown 输出**：清晰的分段输出，包含 AI 升级提示
- 📋 **一键复制**：方便复制 AI 提示到 AI 编程助手

### 技术栈

- **前端**：Next.js (App Router) + TypeScript + TailwindCSS
- **后端**：Next.js API Routes
- **外部 API**：GitHub REST API
- **AI**：OpenAI 兼容 API

## 项目架构与工作流

### 核心工作流程

1. **用户提交表单** → 输入仓库地址和版本范围
2. **获取 GitHub releases** → 收集版本发布信息
3. **筛选版本范围内的 releases** → 确定需要分析的版本区间
4. **调用 GitHub compare API** → 获取版本间的代码差异
5. **收集 commit 和 PR 信息** → 提取详细的变更记录
6. **提取重要 commits** → 智能筛选关键变更
7. **整理结构化数据** → 格式化输入给 AI
8. **发送给 AI 生成** → 生成升级指南
9. **返回结果给前端展示** → 呈现给用户

### AI 输出格式

生成的升级指南包含以下部分：

- `# Upgrade Summary` - 总体升级摘要
- `## Breaking Changes` - 破坏性变更
- `## API Changes` - API 变更
- `## Migration Steps` - 迁移步骤
- `## Important Notes` - 重要提示
- `## AI Upgrade Prompt` - 可复制到 AI 助手的提示词

## 快速使用指南

### 1. 安装依赖

```bash
pnpm install
```

### 2. 配置环境变量

```bash
cp .env.example .env
```

编辑 `.env` 文件，填入你的配置：

```env
GITHUB_TOKEN=your_github_token
OPENAI_API_KEY=your_openai_api_key
OPENAI_BASE_URL=your_openai_base_url  # 可选
```

### 3. 启动开发服务器

```bash
pnpm dev
```

打开浏览器访问 [http://localhost:3000](http://localhost:3000)

### 4. 使用步骤

1. 在首页输入 GitHub 仓库（格式：`owner/repo`）
2. 输入起始版本和目标版本，例如：`17.0.0` → `18.0.0`
3. 点击"Generate Upgrade Guide"
4. 等待分析完成，查看生成的升级指南
5. 复制 AI Upgrade Prompt 部分到你的 AI 编程助手

## 未来发展规划

目前 updateWhat 还是一个 MVP 版本，我们有很多 exciting 的计划来让它变得更加强大和易用！

### 第一阶段：创建 OpenClaw Skill

我们计划将 updateWhat 封装成一个 **OpenClaw Skill**，让用户可以直接通过 OpenClaw AI 助手来使用升级指南生成功能，而不需要打开网页。

#### Skill 功能设计

- **自然语言交互**：用户可以用自然语言描述需求，例如："帮我生成 React 17 到 18 的升级指南"
- **上下文记忆**：记住用户常用的仓库和升级模式
- **智能推荐**：根据用户的项目依赖，主动推荐需要升级的库
- **一键应用**：直接将升级指南应用到用户的项目中

#### Skill 架构

```
updateWhat-skill/
├── SKILL.md              # Skill 描述文档
├── scripts/
│   ├── generate-guide.sh # 生成升级指南的脚本
│   └── parse-input.sh    # 解析用户输入
├── prompts/
│   └── system-prompt.md  # 系统提示词
└── tests/
    └── test-cases.md     # 测试用例
```

### 第二阶段：开发 MCP 服务

更进一步，我们计划开发一个 **MCP (Model Context Protocol) 服务**，让任何兼容 MCP 的 AI 助手都可以使用 updateWhat 的功能。

#### MCP 服务特性

- **标准化接口**：遵循 MCP 协议，支持工具调用和资源访问
- **多平台兼容**：可以在 Claude Desktop、ChatGPT 等平台使用
- **实时数据**：实时从 GitHub 获取最新的版本信息
- **缓存优化**：智能缓存常用仓库的分析结果

#### MCP 工具设计

```typescript
// 工具 1: 生成升级指南
{
  name: "generate_upgrade_guide",
  description: "生成 GitHub 仓库两个版本之间的升级指南",
  inputSchema: {
    type: "object",
    properties: {
      owner: { type: "string", description: "仓库所有者" },
      repo: { type: "string", description: "仓库名称" },
      fromVersion: { type: "string", description: "起始版本" },
      toVersion: { type: "string", description: "目标版本" }
    },
    required: ["owner", "repo", "fromVersion", "toVersion"]
  }
}

// 工具 2: 获取最新版本
{
  name: "get_latest_version",
  description: "获取 GitHub 仓库的最新版本信息",
  inputSchema: {
    type: "object",
    properties: {
      owner: { type: "string", description: "仓库所有者" },
      repo: { type: "string", description: "仓库名称" }
    },
    required: ["owner", "repo"]
  }
}

// 工具 3: 检查依赖更新
{
  name: "check_dependency_updates",
  description: "检查 package.json 中依赖的更新",
  inputSchema: {
    type: "object",
    properties: {
      packageJsonPath: { type: "string", description: "package.json 文件路径" }
    },
    required: ["packageJsonPath"]
  }
}
```

### 第三阶段：增强功能与生态

在基础功能完善后，我们计划添加更多高级功能：

#### 1. 多语言支持
- 支持中文、英文等多种语言的升级指南生成
- 智能识别仓库主要语言，自动适配输出语言

#### 2. 批量升级支持
- 支持一次性检查多个依赖的更新
- 生成综合的升级计划和优先级建议

#### 3. 升级风险评估
- 基于变更范围和社区反馈，评估升级风险等级
- 提供"安全升级"、"谨慎升级"、"暂缓升级"等建议

#### 4. 社区贡献集成
- 集成社区分享的升级经验和踩坑记录
- 支持用户提交和分享自己的升级指南

## 技术亮点与创新

### 1. 智能 Commit 筛选

我们不是简单地把所有 commits 都扔给 AI，而是通过以下规则智能筛选：

- **语义化提交**：识别 `fix:`、`feat:`、`BREAKING CHANGE:` 等提交信息
- **文件路径分析**：重点关注核心源码文件的变更
- **PR 关联**：优先处理关联了 Issue 的 PR
- **影响力评估**：根据变更文件数量和代码行数评估影响力

### 2. 结构化 AI 提示词

我们设计了精心结构化的提示词，确保 AI 输出格式一致且信息完整：

```markdown
你是一个专业的软件升级顾问。请根据以下 GitHub 仓库的版本变更信息，生成一份结构化的升级指南。

要求：
1. 重点突出破坏性变更
2. 提供清晰的迁移步骤
3. 使用 Markdown 格式

仓库信息：
- 所有者：{{owner}}
- 仓库：{{repo}}
- 版本范围：{{fromVersion}} → {{toVersion}}

变更信息：
{{changes}}

请生成升级指南...
```

### 3. 速率限制与缓存策略

为了保证服务稳定性，我们实现了：

- **基于 IP 的速率限制**：防止滥用
- **智能缓存**：缓存常用仓库的分析结果
- **渐进式加载**：先展示概要，再加载详细信息

## 实践案例

### 案例 1：React 17 → 18 升级

假设你要将项目从 React 17 升级到 18：

1. 输入仓库：`facebook/react`
2. 版本范围：`17.0.0` → `18.0.0`
3. 点击生成
4. 获得包含以下内容的升级指南：
   - 并发渲染特性介绍
   - 自动批处理变化
   - 新的 Suspense 行为
   - 破坏性变更清单
   - 逐步迁移指南

### 案例 2：Vue 2 → 3 升级

对于大型 Vue 项目升级：

1. 输入仓库：`vuejs/core`
2. 版本范围：`2.6.0` → `3.0.0`
3. 获得 Composition API 迁移指南
4. 获得 Options API 兼容性说明
5. 获得构建工具配置变更建议

## 总结与展望

**updateWhat** 项目旨在解决开发者在版本升级时的痛点，利用 AI 技术让这个过程变得更加智能和高效。

我们的发展路线图：

1. **短期**：完成 OpenClaw Skill 开发，实现自然语言交互
2. **中期**：推出 MCP 服务，让更多 AI 平台可以使用
3. **长期**：构建完整的升级生态，包括社区分享、风险评估等高级功能

我们相信，通过自动化和智能化的工具，可以让开发者将更多时间专注于创造价值，而不是繁琐的版本升级工作。

如果你对这个项目感兴趣，欢迎关注我们的进展！也欢迎贡献代码、提出建议，一起让 updateWhat 变得更好！🚀

---

**本文由 OpenClaw AI 助手协助创建，展示了 updateWhat 项目的愿景和发展规划。** 👨‍🔧🍄⭐
