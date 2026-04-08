# AGENTS.md

---

## 1. 系统架构

### 1.1 目录结构

```
PersonalWiki/
├── ObsidianRaw/          # 原始资料存储目录（只读）
│   ├── 00_Inbox/         # 收件箱（临时，不处理）
│   ├── 01_Projects/      # 项目（行动导向，不处理）
│   ├── 02_Areas/         # 领域（持续维护，不处理）
│   ├── 03_Resources/     # 资源 ← 唯一可复用的知识来源
│   └── 04_Archive/       # 归档（历史记录，不处理）
├── Wiki/                 # 知识库主目录（由 Agent 维护）
│   ├── Index.md          # 知识库目录
│   ├── Log.md            # 操作日志
│   ├── Concepts/         # 概念页面
│   ├── Entities/         # 实体页面
│   ├── Sources/          # 原始资料摘要
│   └── Outputs/          # 查询产出
├── .claude/skills/       # Claude Skills 定义
├── AGENTS.md             # 本文件（完整协作规范）
└── CLAUDE.md             # 引用 AGENTS.md
```

### 1.2 摄取范围

**仅处理 `ObsidianRaw/03_Resources/` 目录。**

PARA 结构中，各目录的定位不同：
- `00_Inbox/` — 临时收件箱，待分类
- `01_Projects/` — 当前项目，行动导向，有时效性
- `02_Areas/` — 持续责任区域，需主动维护
- `03_Resources/` — 未来可能有用的参考资料，**可复用知识**
- `04_Archive/` — 已完成或不再活跃的内容

Wiki 的目标是构建可复用的知识库，因此只从 Resources 目录摄取。Projects 和 Areas 的内容具有行动属性，不属于可复用知识范畴。

### 1.3 核心工作流

| 工作流 | Skill | 触发场景 |
|--------|-------|----------|
| **摄取** | `ingesting-resources` | 处理 Resources 新文件 |
| **查询** | `querying-wiki` | 用户提问、知识检索 |
| **健康检查** | `checking-wiki-health` | 检测 Wiki 完整性 |
| **同步检测** | `detecting-resources-sync` | 检测新文件、变更文件 |

---

## 2. qmd MCP 工具

本地 Markdown 搜索引擎，支持混合检索（词汇+向量）。

**搜索**：
```javascript
// 关键词
mcp__qmd__query({ searches: [{ type: "lex", query: "韧性" }], limit: 5 })

// 混合（推荐）
mcp__qmd__query({
  searches: [
    { type: "lex", query: "critical infrastructure" },
    { type: "vec", query: "如何评估基础设施的韧性" }
  ],
  intent: "学术研究方法论",  // 消歧义
  limit: 5
})
```

**获取**：
```javascript
mcp__qmd__get({ file: "Wiki/Concepts/韧性.md" })              // 按路径
mcp__qmd__get({ file: "#517f11" })                             // 按 docid
mcp__qmd__multi_get({ pattern: "Wiki/Concepts/**/*.md" })      // 批量 glob
```

**维护**：新增或修改文件后执行 `qmd update && qmd embed`。

---

## 3. 协作规范

### 3.1 文件格式

所有文件使用 Markdown 格式，页面开头必须包含 YAML frontmatter：

```yaml
---
title: 页面标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: concept | entity | source | output
tags: [标签1, 标签2]
---
```

### 3.2 Sources 页面 Frontmatter

Sources 页面需要额外包含追踪字段，用于检测文件同步状态：

```yaml
---
title: 页面标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: source
tags: [标签1, 标签2]
source_path: Resources相对路径.md
source_hash: SHA256前8位
source_mtime: ISO8601时间戳
---
```

| 字段 | 说明 |
|------|------|
| `source_path` | Resources 目录中的相对路径 |
| `source_hash` | 文件内容 SHA256 前 8 位，用于检测变更 |
| `source_mtime` | 文件最后修改时间，用于快速筛选 |

**Hash 计算**：

```bash
# macOS / Linux
sha256sum "$file" | cut -c1-8

# Windows PowerShell
(Get-FileHash "$file" -Algorithm SHA256).Hash.Substring(0,8)

# 跨平台（openssl）
openssl dgst -sha256 "$file" | sed 's/.*= //' | cut -c1-8
```

### 3.3 页面引用

页面间使用 `[[wiki-links]]` 格式引用。

### 3.4 命名规范

- 概念页面: `Wiki/Concepts/{概念名}.md`
- 实体页面: `Wiki/Entities/{实体名}.md`
- 来源摘要: `Wiki/Sources/{来源标识}.md`
- 查询产出: `Wiki/Outputs/{日期}-{主题}.md`

---

## 4. 操作日志格式

日志采用可解析格式，便于用 unix 工具检索：

```markdown
## [YYYY-MM-DD] 操作类型 | 标题

- **来源**: 处理的文件路径（如有）
- **影响**: 创建或更新的页面列表
- **备注**: 操作细节（可选）
```

**解析示例**:

- `grep "^## \[" Log.md | tail -5` — 最近5条记录
- `grep "| ingest" Log.md` — 所有摄取操作

---

## 5. 目录页格式

`Wiki/Index.md` 维护所有页面的目录：

```markdown
# 知识库目录

## 概念
- [[Concepts/概念A]] - 一句话摘要
- [[Concepts/概念B]] - 一句话摘要

## 实体
- [[Entities/实体A]] - 一句话摘要

## 来源
- [[Sources/来源A]] - 一句话摘要（来自 Resources 目录）

## 产出
- [[Outputs/产出A]] - 一句话摘要
```

---

## 6. 搜索优先级

当用户提出问题或请求检索信息时，优先使用 qmd MCP 工具：

1. **混合搜索优先**: 同时使用 `lex` 和 `vec` 类型，获得最佳召回
2. **已知路径**: 使用 `mcp__qmd__get` 精确获取
3. **批量需求**: 使用 `mcp__qmd__multi_get` 配合 glob 模式

---

## 7. 响应风格

- 专业、简洁、系统化
- 避免口语化和模糊表达
- 引用来源时使用 `[[wiki-links]]` 格式

---

## 8. 可用 Skills

| Skill | 触发场景 | 功能 |
|-------|---------|------|
| `ingesting-resources` | 摄取新文件、处理 Resources | 生成摘要、提取概念实体、更新索引 |
| `querying-wiki` | 用户提问、知识检索 | 混合检索、综合回答、结果回填 |
| `checking-wiki-health` | 健康检查、状态检测 | 孤立页面、矛盾、缺口、同步状态 |
| `detecting-resources-sync` | 检测新文件、同步状态 | 新增、变更、删除文件检测 |

Skills 位于 `.claude/skills/` 目录，Claude Code 会自动加载。

---

## 9. 典型工作流示例

### 场景 1：首次使用

1. 运行 `detecting-resources-sync` 获取待处理文件列表
2. 调用 `ingesting-resources` 批量摄取
3. 运行 `checking-wiki-health` 验证结果

### 场景 2：日常维护

1. 定期运行 `detecting-resources-sync` 检测变更
2. 按需调用 `ingesting-resources` 更新 Wiki
3. 每周运行 `checking-wiki-health` 保持健康

### 场景 3：知识查询

1. 用户提出问题
2. 调用 `querying-wiki` 检索并回答
3. 评估结果价值，建议是否回填
