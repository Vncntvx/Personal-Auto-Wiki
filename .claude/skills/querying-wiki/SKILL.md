---
name: querying-wiki
description: 使用 qmd CLI 检索知识库（Wiki 目录），综合回答用户问题。当用户提出问题、请求查询/搜索知识库、检索特定概念或实体时使用。
---

# 知识库查询

使用 `qmd` 命令行工具检索 Wiki 知识库。qmd 索引的是 `Wiki/` 目录（Concepts/Entities/Sources/Outputs），不是 ObsidianRaw 原始文件。

## 首次设置

如果是新的 wiki 目录，先添加到 qmd 索引：

```bash
qmd collection add wiki /path/to/Wiki
qmd embed
```

之后每次修改 Wiki 文件后更新索引：

```bash
qmd update && qmd embed
```

## 检索策略

### 混合搜索（默认推荐）

使用 `qmd query`，自动扩展 + 重排序：

```bash
# 自动扩展（单行查询）
qmd query "韧性理论的发展阶段"

# 结构化查询（手动指定 lex/vec）
qmd query $'intent: 韧性理论发展阶段\nlex: 韧性 发展 阶段\nvec: 韧性理论的演变历程和不同发展阶段'
```

### 精确搜索

```bash
# BM25 关键词搜索（无 LLM 扩展）
qmd search "韧性 工程 生态"

# 纯向量相似度搜索
qmd vsearch "如何评估基础设施的韧性"
```

### 获取文档

```bash
# 按路径获取
qmd get Wiki/Concepts/韧性.md

# 获取指定行（从第10行起取20行）
qmd get Wiki/Concepts/韧性.md:10 -l 20

# 批量获取（glob 模式）
qmd multi-get "Wiki/Concepts/**/*.md"

# 批量获取（逗号分隔列表）
qmd multi-get "Wiki/Concepts/韧性.md,Wiki/Concepts/生态韧性.md"
```

### 浏览索引

```bash
qmd ls wiki                # 列出所有文件
qmd ls wiki/Concepts       # 列出子目录
```

### 查询参数

| 参数 | 说明 |
|------|------|
| `-n <num>` | 返回结果数量，默认 5 |
| `--min-score <num>` | 最小相关性分数，默认 0.5 |
| `--full` | 输出完整文档而非片段 |
| `--files` | 只返回匹配文件路径 |
| `--json` | JSON 格式输出 |
| `--explain` | 显示检索分数追踪（调试用） |
| `-C <num>` | 最大重排序候选数，默认 40（降低可提速） |
| `-c wiki` | 限定 wiki collection |

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

## 查询工作流

复杂查询遵循以下流程：

```
查询进度：
- [ ] 步骤 1：执行混合搜索
- [ ] 步骤 2：阅读匹配文档
- [ ] 步骤 3：综合回答
- [ ] 步骤 4：如有长期价值，询问是否回填到 Outputs/
```

**步骤 1：执行混合搜索**

根据问题类型选择策略（见下面的决策树）。

**步骤 2：阅读匹配文档**

使用 `qmd get <file>` 获取完整文档，或 `qmd multi-get <pattern>` 批量获取。

**步骤 3：综合回答**

按上述回答格式输出，标注来源链接。

**步骤 4：回填**

如果查询结果是综述、对比、分析或结论，询问用户是否保存到 `Wiki/Outputs/`。

## 查询决策树

根据问题类型选择搜索策略：

1. **模糊/开放性问题**（"什么是..."、"...有哪些"）
   → `qmd query "自然语言查询"`（自动扩展）

2. **精确术语查询**（已知具体概念名）
   → `qmd search "术语"` 或 `qmd get Wiki/Concepts/术语.md`

3. **语义/描述性查询**（"如何实现..."、"...的方法"）
   → `qmd query $'lex: 关键词\nvec: 问题含义'`

4. **批量浏览**（了解某类别全貌）
   → `qmd multi-get "Wiki/Concepts/**/主题*.md"` 或 `qmd ls wiki/Concepts`

5. **需要精确上下文**（调试、验证）
   → `qmd query --full --explain` 查看完整文档和分数

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

## 查询优化技巧

### 使用 intent 消歧义

```bash
# 不好：太模糊
qmd query "韧性"

# 好：明确意图
qmd query $'intent: 韧性理论发展阶段\nlex: 韧性 发展 阶段\nvec: 韧性理论的演变历程'
```

### 混合搜索

`lex` 捕捉精确术语，`vec` 捕捉语义关联：

```bash
qmd query $'lex: 贝叶斯 网络\nvec: 贝叶斯网络的应用方法'
```

### 提高精确度

```bash
# 提高分数阈值
qmd query "查询" --min-score 0.6

# 限制重排序数量（提速）
qmd query "查询" -C 20
```

## 要避免的反模式

- ❌ 只对原始文件做 `qmd multi-get` 然后全文搜索——应先通过 `qmd query` 缩小范围
- ❌ 在查询中使用 ObsidianRaw 路径——qmd 索引的是 Wiki 目录
- ❌ 忘记 `qmd update && qmd embed`——修改 Wiki 文件后需更新索引，否则会搜索到过期内容
- ❌ 查询结果不标注来源——所有引用必须使用 `[[wiki-links]]` 格式

## 参考

- 常用查询模式：参阅 [query_patterns.md](reference/query_patterns.md)
