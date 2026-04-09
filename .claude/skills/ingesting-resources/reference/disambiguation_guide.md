# 概念/实体消歧指南

## 内容目录

- 消歧流程
- 相似概念检索
- LLM 消歧判断
- 消歧决策执行
- 别名页面格式
- qmd 不可用回退规则

## 术语定义

术语定义见 AGENTS.md。

## 消歧流程

```
提取候选概念/实体
     ↓
qmd 搜索相似概念 (min-score 0.5)
     ↓
有相似概念？
     ├─ 否 → 创建新页面 (canonical: true)
     └─ 是 → LLM 消歧判断
              ↓
         is_same_concept?
              ├─ 是 → merge: 更新已有页面 + 添加 aliases
              │        或 create_alias: 创建重定向页面
              └─ 否 → 创建新页面 (canonical: true)
```

## 相似概念检索

```bash
# 检索相似概念
qmd query $'intent: 概念消歧\nlex: 韧性 resilience\nvec: 系统恢复能力' \
  -c wiki --glob "Concepts/**" -n 5 --min-score 0.5
```

## LLM 消歧判断

当检测到相似概念时，进行语义判断：

```
判断任务：概念消歧

候选概念：{新概念名}
概念定义：{从源文件提取的定义/描述}

已存在的相似概念：
1. {概念A}（相似度: 0.82）
   定义：{从页面提取的定义}

请判断：
1. 候选概念是否与任一已存在概念为同一概念（同义词、不同语言表达、缩写等）
2. 如为同一概念，推荐合并到哪个概念页面
3. 如为不同概念，说明区别

输出格式：
{
  "is_same_concept": true/false,
  "target_concept": "Concepts/韧性" 或 null,
  "action": "merge" | "create_new" | "create_alias",
  "aliases_to_add": ["resilience"],
  "reason": "判断依据"
}
```

## 消歧决策执行

| 决策 | 执行操作 |
|------|----------|
| `merge` | 更新已存在概念页面：添加别名到 aliases、添加来源引用、补充定义 |
| `create_new` | 创建新概念页面，设置 `canonical: true` |
| `create_alias` | 创建轻量别名页面，设置 `canonical: false`，重定向到规范条目 |

## 别名页面格式

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

## qmd 不可用回退规则

1. 从 `Wiki/Index.md` 定位候选概念页
2. 对候选页定义段进行逐页比对
3. 仅在"同义表达高度确定"时执行 merge，否则保守创建新页面并标注待复核
