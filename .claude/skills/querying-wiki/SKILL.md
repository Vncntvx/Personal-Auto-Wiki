---
name: querying-wiki
description: 检索 PersonalWiki 知识库并回答问题，支持混合检索（词汇+语义）。当用户提出问题、请求查询、搜索知识库时使用。
---

# 知识库查询

使用 qmd MCP 工具检索 Wiki 知识库，综合回答用户问题。

## 触发条件

- 用户提出问题
- 用户请求"查询"、"搜索"知识库
- 用户请求检索特定概念或实体

## 检索策略

### 混合搜索（推荐）

同时使用词汇搜索和语义搜索，获得最佳召回：

```javascript
mcp__qmd__query({
  searches: [
    { type: "lex", query: "关键词" },
    { type: "vec", query: "语义描述，描述问题的含义" }
  ],
  intent: "问题意图，帮助消歧义",
  limit: 5
})
```

### 精确获取

已知文件路径时直接获取：

```javascript
// 按路径
mcp__qmd__get({ file: "Wiki/Concepts/韧性.md" })

// 按 docid
mcp__qmd__get({ file: "#517f11" })
```

### 批量获取

```javascript
// glob 模式
mcp__qmd__multi_get({ pattern: "Wiki/Concepts/**/*.md" })

// 多个文件
mcp__qmd__multi_get({ files: ["Concepts/韧性.md", "Concepts/工程韧性.md"] })
```

### 搜索参数

| 参数 | 说明 |
|------|------|
| `limit` | 返回结果数量，默认 5 |
| `minScore` | 最小相关性分数，默认 0.5 |
| `intent` | 问题意图，帮助消歧义 |

## 回答格式

### 结构

1. **直接回答**：针对问题的核心答案
2. **来源引用**：使用 `[[wiki-links]]` 格式标注
3. **相关概念**：提及相关的概念和实体

### 引用格式

- 概念：`[[Concepts/概念名]]`
- 实体：`[[Entities/实体名]]`
- 来源：`[[Sources/来源名]]`
- 带别名：`[[Entities/Holling|C.S. Holling]]`

## 结果回填

有价值的查询结果建议保存到 `Wiki/Outputs/`。

**值得保存的结果类型**：
- 综述：跨多个来源的综合分析
- 对比：概念、实体、方法的对比表
- 分析：发现隐藏联系或洞察
- 结论：需要记录的推理结果

**回填格式**：

```yaml
---
title: 查询主题
created: YYYY-MM-DD
type: output
query: 原始问题
sources: [[Sources/A]], [[Sources/B]]
---
```

**回填时机**：回答结束后，询问用户是否保存。

## 示例

**用户问题**：韧性理论有哪些发展阶段？

**检索**：

```javascript
mcp__qmd__query({
  searches: [
    { type: "lex", query: "韧性 发展 阶段" },
    { type: "vec", query: "韧性理论的演变历程和不同发展阶段" }
  ],
  intent: "韧性理论发展阶段",
  limit: 5
})
```

**回答**：

韧性理论经历了三个发展阶段（参考 [[Sources/韧性]]）：

1. **工程韧性**（第一阶段）：单一稳态恢复能力 [[Concepts/工程韧性]]
2. **生态韧性**（第二阶段）：多稳态扰动吸收能力 [[Concepts/生态韧性]]
3. **演进韧性**（第三阶段）：系统变化、适应和转化能力 [[Concepts/演进韧性]]

主要贡献者包括 [[Entities/Holling|C.S. Holling]]、[[Entities/Folke|Carl Folke]] 等。
