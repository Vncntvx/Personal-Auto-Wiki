---
name: querying-wiki
description: 使用 qmd CLI 检索知识库（Wiki 目录），综合回答用户问题。当用户提出问题、请求查询/搜索知识库、检索特定概念或实体时使用。
---

# 知识库查询

使用 `qmd` 命令行工具检索 Wiki 知识库。qmd 索引的是 `Wiki/` 目录（Concepts/Entities/Sources/Outputs），不是 ObsidianRaw 原始文件。

## 参考资料

- 查询模式与优化技巧：参阅 [query_patterns.md](reference/query_patterns.md)
- 页面内容模板：参阅 [page_templates.md](../ingesting-resources/reference/page_templates.md)

## 术语定义

术语定义见 AGENTS.md。本 Skill 特有阈值：相似输出检测默认 `0.75`；日志类型使用 `query`。

## 检索策略

### 检索优先级

1. 先读 `Wiki/Index.md`，定位候选主题与页面范围
2. 优先读取稳定知识页（`Concepts/`、`Entities/`）和已有分析输出页（`Outputs/`）
3. 使用 `qmd query` 做混合搜索，补充召回范围
4. 当稳定知识不足时，再展开阅读 `Sources/`
5. `qmd` 不可用时，退化到"Index 导航 + 文件直读"模式

如果回答必须拼接多个 `Sources/` 页面，说明当前 Wiki 缺少可复用的综合结果，应明确指出知识缺口，并评估是否回填到 `Outputs/`。

### 混合搜索（默认推荐）

```bash
# 自动扩展
qmd query "韧性理论的发展阶段"

# 结构化查询
qmd query $'intent: 韧性理论发展阶段\nlex: 韧性 发展 阶段\nvec: 韧性理论的演变历程'
```

### 精确搜索与获取

```bash
qmd search "韧性 工程 生态"              # BM25 关键词
qmd vsearch "如何评估基础设施的韧性"      # 纯向量
qmd get Wiki/Concepts/韧性.md            # 按路径获取
qmd get Wiki/Concepts/韧性.md:10 -l 20   # 指定行范围
qmd multi-get "Wiki/Concepts/**/*.md"    # 批量获取（glob）
qmd ls wiki/Concepts                     # 浏览索引
```

更多查询模式与优化技巧见 [query_patterns.md](reference/query_patterns.md)。

## 回答格式

1. **直接回答**：针对问题的核心答案
2. **来源引用**：使用 `[[wiki-links]]` 格式标注
3. **相关概念**：提及相关的概念和实体

引用格式：`[[Concepts/概念名]]`、`[[Entities/实体名]]`、`[[Sources/来源名]]`、`[[Entities/Holling|C.S. Holling]]`

## 查询工作流

```
查询进度：
- [ ] 步骤 0：读取 Index 导航
- [ ] 步骤 1：相似问题检索
- [ ] 步骤 2：执行混合搜索
- [ ] 步骤 3：阅读匹配文档
- [ ] 步骤 4：综合回答
- [ ] 步骤 5：识别知识缺口
- [ ] 步骤 6：回填与日志留痕
```

**步骤 0：读取 Index 导航**

先读取 `Wiki/Index.md`，定位候选页面范围。

**步骤 1：相似问题检索**

检测是否已有相似问题的输出：

```
用户提问 → 计算 query_hash → 检查 Outputs/ 是否有相同 query_hash？
  → 是 → 提示用户查看已有输出
  → 否 → 检索相似 Outputs (min-score 0.75)
    → 有相似 → 提示用户选择：查看已有/继续查询
    → 无 → 继续步骤 2
```

query_hash 计算方法见 [query_patterns.md](reference/query_patterns.md)。

**步骤 2：执行混合搜索**

根据问题类型选择策略（见查询决策树）。优先读取稳定知识页与分析输出页。

**步骤 3：阅读匹配文档**

使用 `qmd get <file>` 获取完整文档，或 `qmd multi-get <pattern>` 批量获取。

**步骤 4：综合回答**

按回答格式输出，标注来源链接。

**步骤 5：识别知识缺口**

如果回答依赖多个 `Sources/` 页面拼接，或现有页面只能提供零散信息：
- 明确指出当前 Wiki 缺少能直接回答该主题的稳定知识页或分析输出页
- 判断本次回答是否已形成可复用分析结果

**步骤 6：回填与日志留痕**

仅在用户明确同意时，才写入 `Wiki/Outputs/`。满足以下任一条件时必须追加 `Wiki/Log.md`（`query` 类型）：形成可复用输出并回填 `Outputs/`；明确识别并确认知识缺口；对既有输出执行 merge/replace 决策。

## 查询决策树

1. **模糊/开放性问题**（"什么是..."、"...有哪些"）
   → `qmd query "自然语言查询"`

2. **精确术语查询**（已知具体概念名）
   → `qmd search "术语"` 或 `qmd get Wiki/Concepts/术语.md`

3. **语义/描述性查询**（"如何实现..."、"...的方法"）
   → `qmd query $'lex: 关键词\nvec: 问题含义'`

4. **批量浏览**（了解某类别全貌）
   → `qmd multi-get "Wiki/Concepts/**/主题*.md"` 或 `qmd ls wiki/Concepts`

5. **需要精确上下文**（调试、验证）
   → `qmd query --full --explain`

## 结果回填

有价值的查询结果建议保存到 `Wiki/Outputs/`。

**值得保存的结果类型**：综述（跨来源分析总结）、对比（概念/方法对比表）、分析（发现隐藏联系）、结论（推理结果）、缺口补全。

### 相似性检测（去重）

保存前必须执行相似性检测：

```
用户同意保存 → 计算 query_hash → 检查是否有相同 query_hash
  → 是 → 提示「完全重复」，建议更新现有输出或跳过
  → 否 → 语义相似性搜索
    → 相似度 ≥ 0.75 → 提示用户选择：合并/替代/独立创建
    → 相似度 < 0.75 → 直接创建新输出
```

query_hash 计算方法见 [query_patterns.md](reference/query_patterns.md)。Outputs 页面 frontmatter 格式见 [page_templates.md](../ingesting-resources/reference/page_templates.md)。

**回填后的处理原则**：
- 如该输出已成为某主题的稳定综合结果，应在后续维护中考虑整合回稳定知识页
- 如后续来源改变了该输出的结论，应在同步或健康检查中将其识别为待重审对象

## 要避免的反模式

- ❌ 跳过 `Index.md` 直接全库检索
- ❌ 只对原始文件做 `qmd multi-get` 然后全文搜索
- ❌ 在查询中使用 ObsidianRaw 路径
- ❌ 修改 Wiki 文件后忘记 `qmd update && qmd embed`
- ❌ 查询结果不标注来源

## 初始化

若项目第一次初始化或 `qmd` 可用但尚未将 `Wiki/` 添加到索引（或需管理已索引的文件夹），可以使用 `qmd` 的 collection 子命令管理索引文件夹：`qmd collection add|list|remove|rename|show path`。path 选项可以是相对路径（相对于项目根目录）或绝对路径。


## 错误处理

| 错误类型 | 处理方式 |
|---------|---------|
| `qmd` 不可用 | 退化到 `Index.md` + 文件直读，标注"检索模式：降级" |
| 检索结果为空 | 放宽阈值后重试；仍为空则返回"当前知识库暂无直接答案" |
| 相似输出冲突 | 要求用户在 merge / replace / create 中明确决策后再写入 |
