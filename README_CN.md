
<h1 align="center">Personal Auto Wiki</h1>

<p align="center" style="color: #888; font-size: 0.95em; margin-top: -0.5em; margin-bottom: 0.5em;">
   <b>中文</b> | <a href="README.md"> English</a>
</p>

个人知识库管理，通过 Skills 封装核心工作流，将可复用知识资料自动转化为结构化的 Wiki 知识库。结合 `qmd` MCP 搜索引擎，实现知识的自动摄取、智能查询和健康维护，实现知识的长期积累和高效检索。

> 项目博客：<https://blog.wenjiexu.site/zh/posts/llm-personal-wiki/>

## 系统概览

### 核心能力

- 🔄 **自动同步检测** — 发现 Resources 目录的新增、变更和删除文件
- 📥 **智能知识摄取** — 从原始资料中提取概念、实体，生成摘要并建立索引
- 🔍 **混合检索** — 结合词汇搜索和向量搜索，精准召回相关知识
- 📊 **健康检查** — 检测孤立页面、概念缺口、矛盾标注和同步状态

---

## 架构设计

### 目录结构

```text
PersonalWiki/
├── ObsidianRaw/                  # 原始资料存储（PARA 方法）
│   ├── 00_Inbox/                 # 收件箱（临时，不处理）
│   ├── 01_Projects/              # 项目（行动导向，不处理）
│   ├── 02_Areas/                 # 领域（持续维护，不处理）
│   ├── 03_Resources/             # 资源 ← 唯一摄取来源
│   └── 04_Archive/               # 归档（历史记录，不处理）
├── Wiki/                         # 知识库（由 AI Agent 维护）
│   ├── Index.md                  # 知识库目录索引
│   ├── Log.md                    # 操作日志（可解析格式）
│   ├── Concepts/                 # 概念页面（理论知识）
│   ├── Entities/                 # 实体页面（人物、组织、工具等）
│   ├── Sources/                  # 来源摘要（追踪原始文件状态）
│   └── Outputs/                  # 查询产出（用户问答记录）
├── .claude/skills/               # Skills 定义
│   ├── ingesting-resources/      # 知识摄取工作流
│   ├── querying-wiki/            # 知识查询工作流
│   ├── checking-wiki-health/     # 健康检查工作流
│   └── detecting-resources-sync/ # 同步检测工作流
├── AGENTS.md                     # 完整协作规范
└── README.md                     # 本文件
```

### 摄取范围

**仅处理 `ObsidianRaw/03_Resources/` 目录。**

| 目录 | 定位 | 是否处理 |
|------|------|----------|
| `00_Inbox/` | 临时收件箱，待分类 | ❌ |
| `01_Projects/` | 当前项目，行动导向 | ❌ |
| `02_Areas/` | 持续责任区域 | ❌ |
| `03_Resources/` | 可复用知识资料 | ✅ 唯一来源 |
| `04_Archive/` | 已完成/不活跃内容 | ❌ |

### 跳过规则

以下内容不会被摄取：

- `Personal/` 目录（敏感信息）
- 仅包含图片的文件（无文本内容）
- 购物清单/配置清单（无知识价值）

---

## 快速开始

### 1. 检测新文件

```text
检测新文件
```

输出 Resources 目录与 Wiki/Sources 的同步状态：

- **新增文件** — 待摄取
- **变更文件** — 需更新
- **删除文件** — 需标记

### 2. 摄取知识

```bash
# 摄取单个文件
摄取 Academic/NewTopic.md

# 摄取整个目录
摄取 Academic/Research Notes/Theory/

# 摄取所有新增文件
摄取所有新增文件
```

### 3. 查询知识

直接提问即可，系统会自动检索并回答：

```text
韧性理论有哪些发展阶段？
贝叶斯网络的应用场景有哪些？
```

### 4. 健康检查

```text
健康检查
```

输出 Wiki 状态报告：孤立页面、概念缺口、矛盾标注、未同步文件。

---

## 工作流指南

### 首次使用

```text
1. 检测新文件        → 查看待处理文件列表
2. 摄取所有新增文件   → 批量构建知识库
3. 健康检查          → 验证构建结果
```

### 日常维护

```text
1. 定期检测新文件     → 发现变更
2. 摄取指定文件       → 更新知识库
3. 每周健康检查       → 保持知识库健康
```

### 知识查询

```text
1. 直接提问          → 获得综合回答
2. 如有长期价值      → 建议保存到 Outputs/ 作为永久记录
```

---

## 可用命令

| 触发词 | 对应 Skill | 功能 |
|--------|-----------|------|
| `同步并处理`、`完整同步`、`检测并处理` | `syncing-wiki` | 完整工作流（推荐） |
| `检测新文件`、`同步状态` | `detecting-resources-sync` | 仅检测变更状态 |
| `摄取`、`处理新文件`、`更新 Wiki` | `ingesting-resources` | 摄取知识到 Wiki |
| `健康检查`、`检查状态`、`诊断` | `checking-wiki-health` | 检查 Wiki 健康状态 |
| 直接提问 | `querying-wiki` | 检索知识并回答 |

---

## 技术规范

### 文件格式

所有页面使用标准 YAML frontmatter：

```yaml
---
title: 页面标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: concept | entity | source | output
tags: [标签1, 标签2]
---
```

### Sources 追踪字段

来源摘要页面包含额外字段用于同步检测：

```yaml
source_path: Academic/Research Notes/Theory/韧性.md
source_hash: 45ea3a15        # SHA256 前 8 位
source_mtime: 2026-04-08T15:00:05Z
```

### 页面引用

页面间使用 `[[wiki-links]]` 格式互相引用。

### 命名规范

| 类型 | 路径 | 示例 |
|------|------|------|
| 概念 | `Wiki/Concepts/{概念名}.md` | `Concepts/韧性.md` |
| 实体 | `Wiki/Entities/{实体名}.md` | `Entities/贝叶斯.md` |
| 来源 | `Wiki/Sources/{来源标识}.md` | `Sources/韧性理论笔记.md` |
| 产出 | `Wiki/Outputs/{日期}-{主题}.md` | `Outputs/2026-04-08-韧性分析.md` |

---

## 搜索引擎

使用 `qmd` MCP 工具进行本地 Markdown 搜索，支持：

- **词汇搜索** (`lex`) — 精确匹配关键词
- **向量搜索** (`vec`) — 语义相似度匹配
- **混合搜索** — 同时使用两种策略（推荐）
- **精确获取** — 通过路径或 docid 获取完整内容

---

## 依赖工具

- AI Agent 运行环境，例如 Claude Code, Codex等
- qmd CLI 和 MCP 工具，本地 Markdown 搜索引擎
- openssl，用于计算文件哈希

---

## 参考文档

- [Karpathy: LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — original inspiration
- [PARA method](https://fortelabs.com/blog/para/) — organization framework for raw sources
- [Obsidian](https://obsidian.md/) — local Markdown knowledge browser
- [qmd](https://github.com/nickthecat/qmd) — local Markdown search engine