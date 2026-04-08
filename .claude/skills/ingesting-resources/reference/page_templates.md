# 页面内容模板

## Sources 页面模板

```markdown
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
---

# 概念名称

## 定义

[简洁的定义，1-2 句话]

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
```

## Entities 页面模板

```markdown
---
title: 实体名称
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: entity
tags: [标签1, 标签2]
---

# 实体名称

## 简介

[实体简介，1-2 句话]

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | 工具/人物/组织 |
| 领域 | 相关领域 |

## 相关概念

- [[Concepts/相关概念A]]

## 来源

- [[Sources/来源A]]
```

## Outputs 页面模板

```markdown
---
title: 查询主题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: output
query: 原始问题
sources: [[Sources/A]], [[Sources/B]]
tags: [标签1, 标签2]
---

# 查询主题

## 问题

[原始问题]

## 回答

[综合回答内容]

## 来源

- [[Sources/A]]
- [[Sources/B]]

## 相关概念

- [[Concepts/概念A]]
```
