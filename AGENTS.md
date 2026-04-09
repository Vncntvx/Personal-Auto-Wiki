# AGENTS.md

---

## 1. 系统架构

```
PersonalWiki/
├── ObsidianRaw/          # 原始资料（只读）
│   ├── 00_Inbox/         #   临时收件箱（不处理）
│   ├── 01_Projects/      #   项目（行动导向，不处理）
│   ├── 02_Areas/         #   领域（持续维护，不处理）
│   ├── 03_Resources/     #   资源 ← 唯一可摄取的知识来源
│   └── 04_Archive/       #   归档（不处理）
├── Wiki/                 # 知识库主目录（由 Agent 维护）
│   ├── Index.md          #   知识库目录
│   ├── Log.md            #   操作日志
│   ├── Concepts/         #   概念页面
│   ├── Entities/         #   实体页面
│   ├── Sources/          #   原始资料摘要
│   └── Outputs/          #   查询产出
├── .claude/skills/       # Claude Skills 定义
├── AGENTS.md             # 本文件
└── CLAUDE.md             # 引用本文件
```

**摄取范围**：仅处理 `ObsidianRaw/03_Resources/`。其余 PARA 目录（Inbox/Projects/Areas/Archive）具有行动属性或时效性，不属于可复用知识范畴。

---

## 2. 全局术语

为避免跨技能与文档歧义，统一使用以下术语：

| 术语 | 定义 | 边界 |
|------|------|------|
| Resources 原始文件 | `ObsidianRaw/03_Resources/` 下的 Markdown 文件 | 仅读取，不直接改写 |
| Sources 页面 | `Wiki/Sources/` 下的来源摘要页（`type: source`） | 用于来源追踪与摘要沉淀 |
| 稳定知识页 | `Wiki/Concepts/` 与 `Wiki/Entities/` 页面 | 查询优先复用 |
| 分析输出页 | `Wiki/Outputs/` 页面（`type: output`） | 查询产出的可复用综合 |
| 同步状态 | 新增 (`new`) / 变更 (`modified`) / 删除 (`deleted`) / 同步 (`synced`) | 重命名 (`renamed`) 由删除+新增 hash 匹配推断 |
| 认知质量 | 页面结构完整度与知识深度 | 量化评分见 checking-wiki-health skill |
| 当前综合 | 稳定知识页中基于来源的综合说明段落 | 新来源改变理解时必须重写 |
| 摄取 | 将 Resources 原始文件转化为 Wiki 页面 | 由 ingesting-resources skill 执行 |
| 回填 | 将查询结果写入 `Wiki/Outputs/` | 仅在用户同意时执行 |
| 矛盾标注 | `> [!warning] 矛盾标注` callout 块 | 标注不同来源间的矛盾观点 |

> 术语一致性要求：后续所有 Skills、Reference、日志与报告应优先使用上述术语，不混用同义表达。

---

## 3. 知识演化原则

本知识库是持续维护的 **compounding wiki**，每次操作都应让整体理解更完整、更可复用。

1. **先综合，后归档**：`Sources/` 保留来源摘要与出处；稳定知识页负责沉淀可复用知识；分析输出页负责沉淀查询中的高价值综合。
2. **知识增量必须传播**：新来源不只创建 Sources 页面，还应检查并更新受影响的稳定知识页与分析输出页。如改变已有理解，必须重写"当前综合"部分，而非仅追加引用。
3. **查询也是知识生产**：跨来源综述、对比分析、阶段性结论不应仅停留在对话中。如具长期价值，应回填到分析输出页。当查询暴露出稳定知识页或分析输出页长期缺失时，应视为知识缺口。
4. **健康检查兼顾结构与认知**：不仅检查缺页、孤立页、缺链、未同步文件，还要识别过时结论、浅层综合、长期未处理矛盾、应沉淀但未沉淀的输出。
5. **Schema 与知识库共演**：Schema（本文件、Skills、Reference）不是静态规范，应随知识库增长和使用反馈持续迭代。变更必须记录到 `CHANGELOG.md`，重大变更同步更新 `VERSION`。

---

## 4. 工作流编排

| 工作流 | Skill | 触发场景 |
|--------|-------|----------|
| 完整同步 | `syncing-wiki` | 检测并处理所有变更 |
| 摄取 | `ingesting-resources` | 处理 Resources 新文件 |
| 查询 | `querying-wiki` | 用户提问、知识检索 |
| 健康检查 | `checking-wiki-health` | 检测 Wiki 完整性 |
| 同步检测 | `detecting-resources-sync` | 仅检测变更状态 |

**编排顺序（强约束）**：检测（detecting-resources-sync）→ 摄取（ingesting-resources）→ 验证（checking-wiki-health）→ 查询（querying-wiki）→ 留痕（Log.md）。保证"先检测、后处理、再验证、最后留痕"的可追溯闭环。

**推荐**：日常维护使用 `syncing-wiki` 一次性完成检测和处理。

---

## 5. 协作规范

### 文件格式

所有文件使用 Markdown，页面开头必须包含 YAML frontmatter：

```yaml
---
title: 页面标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: concept | entity | source | output
tags: [标签1, 标签2]
---
```

Sources 页面需额外追踪字段（docid、source_path、source_hash、source_mtime、aliases），稳定知识页可含消歧与统计字段，分析输出页可含去重字段。详见 ingesting-resources skill 的 reference 文件：
- Frontmatter 模板与字段说明：`frontmatter_template.md`
- 页面内容模板：`page_templates.md`

### 页面引用

页面间使用 `[[wiki-links]]` 格式引用。

### 命名规范

- 概念页面：`Wiki/Concepts/{概念名}.md`
- 实体页面：`Wiki/Entities/{实体名}.md`
- 来源摘要：`Wiki/Sources/{来源标识}.md`
- 查询产出：`Wiki/Outputs/{日期}-{主题}.md`

---

## 6. 日志与索引

**日志**（`Wiki/Log.md`）采用可解析格式：

```markdown
## [YYYY-MM-DD] 操作类型 | 标题

- **来源**: 处理的文件路径（如有）
- **影响**: 创建或更新的页面列表
- **备注**: 操作细节（可选）
```

操作类型枚举：`ingest` | `query` | `lint` | `sync`

> Schema 变更（AGENTS.md、Skills、Reference）记录到项目根目录 `CHANGELOG.md`，不在 Log.md 中记录。

解析示例：`grep "^## \[" Log.md | tail -5`、`grep "| ingest" Log.md`

**索引**（`Wiki/Index.md`）列出所有页面及其一句话摘要，按概念/实体/来源/产出分类。

---

## 7. 搜索优先级

1. 先读 `Wiki/Index.md` 定位候选页面范围
2. 优先检索稳定知识页与分析输出页
3. 混合搜索（`qmd query`）增强召回
4. 精确读取（`qmd get` / `qmd multi-get`）
5. 稳定知识不足时再展开 `Sources/`
6. `qmd` 不可用时退化到 Index 导航 + 文件直读

如需拼接多个 Sources 页面才能回答，视为知识缺口；结果具长期价值时优先回填到分析输出页。

---

## 8. 可用 Skills

| Skill | 触发场景 | 功能 |
|-------|---------|------|
| `syncing-wiki` | 同步并处理、完整同步 | 完整工作流：检测→确认→处理→验证→日志→报告 |
| `ingesting-resources` | 摄取新文件、处理 Resources | 生成摘要、提取概念实体、更新相关页面与索引 |
| `querying-wiki` | 用户提问、知识检索 | 混合检索、综合回答、识别知识缺口、结果回填 |
| `checking-wiki-health` | 健康检查、状态检测 | 孤立页面、矛盾、缺口、同步状态、认知质量 |
| `detecting-resources-sync` | 检测新文件、同步状态 | 仅检测变更状态，不自动处理 |

Skills 位于 `.claude/skills/` 目录，Claude Code 按需自动加载。

---

## 9. 安全与写入边界

### 角色分工

| 角色 | 权限 | 禁止项 |
|------|------|--------|
| User | 定义目标、确认范围、审批高影响决策 | — |
| Agent | 在允许范围内执行读写、检索、同步、留痕 | 越权修改未授权目录 |
| Skills | 提供任务级流程规范与模板 | 取代全局架构约束 |

### 写入边界（强约束）

- 允许写入：`Wiki/`、`.claude/skills/`、`AGENTS.md`、`CLAUDE.md`
- 禁止写入：`ObsidianRaw/`（仅只读）、未授权配置/脚本目录
- 若用户仅要求评估/分析，默认只读，不进行任何写入

### 安全与合规

- 不写入凭据、密钥、令牌等敏感信息
- 不在日志中记录隐私原文，仅记录必要摘要与路径
- 高风险操作（批量删除、覆盖）必须先显式确认

---

## 10. 兼容性

- 架构规范版本：见项目根目录 `VERSION` 文件
- Schema 变更记录：见项目根目录 `CHANGELOG.md`
- 默认依赖：`qmd` CLI（混合检索）；不可用时退化到 `Index.md` + 文件直读
- 哈希工具：优先 `openssl`，必要时使用系统原生命令

---

## 11. 响应风格

- 专业、简洁、系统化
- 避免口语化和模糊表达
- 引用来源时使用 `[[wiki-links]]` 格式
- 同步状态术语统一使用：新增 / 变更 / 删除 / 同步 / 重命名（必要时附 machine label）
