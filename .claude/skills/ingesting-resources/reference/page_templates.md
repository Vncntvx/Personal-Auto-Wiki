# 页面内容模板

## 内容目录

- Sources 页面模板
- Concepts 页面模板
- Entities 页面模板
- 别名页面模板
- Outputs 页面模板
- Frontmatter 字段参考

## Sources 页面模板

```markdown
---
title: 页面标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: source
tags: [标签1, 标签2]
docid: renxing_abc123de            # 全局唯一标识符
source_path: Academic/Research Notes/Theory/名词概念/韧性.md
source_hash: a1b2c3d4
source_mtime: 2026-04-09T08:30:00Z
aliases: []                       # 历史路径（文件重命名时更新）
---

# 标题

> 来源：`ObsidianRaw/03_Resources/相对路径.md`

## 摘要

[1-3 句话概括核心内容，让读者快速了解本文要点]

## 核心内容

### 章节1

[按原文结构提炼关键内容]

### 章节2

[保持原文的逻辑层次]

## 关键概念

- [[Concepts/概念A]] - 一句话说明
- [[Concepts/概念B]] - 一句话说明

## 相关实体

- [[Entities/实体A]] - 一句话说明

## 参考文献

[如有引用文献，列出关键文献]
```

## Concepts 页面模板

```markdown
---
title: 概念名称
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: concept
tags: [标签1, 标签2]
canonical: true                  # 是否为规范条目（默认 true）
aliases:                         # 同义词/别名列表
  - 同义词1
  - synonym
canonical_of: null               # 如非规范条目，指向规范条目
stats:                           # 统计字段（可选，健康检查时自动计算）
  char_count: 1250
  source_count: 3
  backlink_count: 5
  quality_score: 0.85
disambiguation:                  # 消歧记录（可选）
  performed: true
  date: YYYY-MM-DD
  candidates: []
  decision: create_new
  reason: "判断依据"
last_reviewed: YYYY-MM-DD        # 最后审核日期（可选）
---

# 概念名称

## 定义

[简洁的定义，1-2 句话]

## 当前综合

[基于现有来源与输出，对该概念当前理解的综合说明]

## 核心特征

- 特征1
- 特征2
- 特征3

## 相关概念

- [[Concepts/相关概念A]]
- [[Concepts/相关概念B]]

## 来源

- [[Sources/来源A]]
- [[Sources/来源B]]

## 相关输出

- [[Outputs/输出A]]
```

## Entities 页面模板

```markdown
---
title: 实体名称
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: entity
tags: [标签1, 标签2]
canonical: true                  # 是否为规范条目（默认 true）
aliases:                         # 同义词/别名列表
  - 别名1
canonical_of: null               # 如非规范条目，指向规范条目
stats:                           # 统计字段（可选）
  char_count: 800
  source_count: 2
  backlink_count: 3
  quality_score: 0.72
---

# 实体名称

## 简介

[实体简介，1-2 句话]

## 当前综合

[该实体在现有来源中的角色、意义或主要结论]

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | 工具/人物/组织 |
| 领域 | 相关领域 |

## 相关概念

- [[Concepts/相关概念A]]

## 来源

- [[Sources/来源A]]

## 相关输出

- [[Outputs/输出A]]
```

## 别名页面模板（轻量重定向）

```markdown
---
title: Resilience Theory
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: concept
tags: [alias]
canonical: false
canonical_of: Concepts/韧性
aliases: [resilience]
---

# Resilience Theory

> 本条目为 [[Concepts/韧性]] 的别名。

参见规范条目：[[Concepts/韧性]]
```

## Outputs 页面模板

```markdown
---
title: 查询主题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: output
query: 原始问题
sources:
  - [[Sources/A]]
  - [[Sources/B]]
tags: [标签1, 标签2]
query_hash: 7f3a2b1c             # query 的 hash，用于快速去重
similarity_check:                # 相似性检测记录
  performed: true
  date: YYYY-MM-DD
  similar_outputs: []            # 相似输出列表（如有）
  decision: create               # create | merge | replace | skip
---

# 查询主题

## 问题

[原始问题]

## 回答

[综合回答内容]

## 结论总结

[本次输出形成的可复用结论、比较框架或分析结果]

## 来源

- [[Sources/A]]
- [[Sources/B]]

## 相关概念

- [[Concepts/概念A]]

## 后续处理

- 是否建议整合回 `Concepts/` 或 `Entities/`
- 是否填补了现有 Wiki 的知识缺口
```

---

Frontmatter 字段详情（Sources 专用字段、Concepts/Entities 消歧字段、统计字段、Outputs 去重字段）见 [frontmatter_template.md](frontmatter_template.md)。
