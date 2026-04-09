---
name: ingesting-resources
description: 摄取 Resources 目录的 Markdown 文件，生成摘要页面、提取概念实体、更新索引。当用户请求"摄取"、"处理新文件"、"更新Wiki"时使用。
---

# 资源摄取

将 Resources 目录的原始文件转化为 Wiki 知识库页面。

## 参考资料

- Frontmatter 模板与字段说明：参阅 [frontmatter_template.md](reference/frontmatter_template.md)
- 页面内容模板：参阅 [page_templates.md](reference/page_templates.md)
- 消歧详细流程：参阅 [disambiguation_guide.md](reference/disambiguation_guide.md)

## 术语定义

术语定义见 AGENTS.md。

## 触发条件

- 用户明确请求摄取特定文件或目录
- `detecting-resources-sync` 检测到新增/变更文件后，用户确认摄取
- 用户请求"处理新文件"、"更新 Wiki"

## 范围限制

仅摄取 `ObsidianRaw/03_Resources/`（见 AGENTS.md）。

## 摄取流程

### 步骤 1：读取源文件

读取指定文件的完整内容，识别结构（标题层级、章节、列表、表格、代码块）。

回答以下问题：
- 这个来源增加了哪些新信息、新视角或新证据
- 它与现有 Wiki 中哪些页面相关
- 它是否改变现有理解，而不只是补充材料

### 步骤 2：生成 Sources 页面

在 `Wiki/Sources/` 创建摘要页面。Frontmatter 格式见 [frontmatter_template.md](reference/frontmatter_template.md)，内容模板见 [page_templates.md](reference/page_templates.md)。

**docid 格式**：`{语义前缀}_{8位十六进制}`，示例：`renxing_a1b2c3d4`。生成规则详见 [frontmatter_template.md](reference/frontmatter_template.md)。

### 步骤 3：提取概念和实体

**概念提取标准**：方法论、理论框架、术语定义，可复用的抽象知识。命名：`Wiki/Concepts/{概念名}.md`

**实体提取标准**：人物、组织、工具、数据库，具体的可命名对象。命名：`Wiki/Entities/{实体名}.md`

### 步骤 3.5：概念/实体消歧检测

对每个提取的概念/实体，执行消歧检测避免重复创建。核心流程：

```
提取候选 → qmd 搜索相似概念 (min-score 0.5)
  → 无相似 → 创建新页面 (canonical: true)
  → 有相似 → LLM 消歧判断
     → 同一概念 → merge 或 create_alias
     → 不同概念 → 创建新页面 (canonical: true)
```

LLM 消歧判断 prompt 模板、决策执行细节、别名页面格式见 [disambiguation_guide.md](reference/disambiguation_guide.md)。

**相似概念检索**：

```bash
qmd query $'intent: 概念消歧\nlex: 韧性 resilience\nvec: 系统恢复能力' \
  -c wiki --glob "Concepts/**" -n 5 --min-score 0.5
```

`qmd` 不可用时回退：从 `Wiki/Index.md` 定位候选，逐页比对定义段，仅在高确定性时执行 merge，否则创建新页面并标注待复核。

### 步骤 4：关联影响分析与综合更新

对每个提取的概念/实体及可能受影响的分析输出页，必须进行影响分析。

**已存在**：添加新引用来源、更新定义或说明、如新来源改变现有理解**必须重写**"当前综合"部分、标注矛盾（如有）、检查相关 `Outputs/` 是否受影响。

**不存在**：创建新页面。

**必须显式回答**：
1. 新来源增加了什么
2. 它影响了哪些稳定知识页与分析输出页
3. 它是否改变了当前对相关概念/实体的理解
4. 它是否制造或缓解了矛盾
5. 它是否暴露出待研究问题或知识缺口

**矛盾标注格式**：

```markdown
> [!warning] 矛盾标注
> 本页面与 [[Sources/另一来源]] 存在矛盾：
> - 本页面观点：...
> - 另一来源观点：...
```

### 步骤 5：更新索引和日志

**Index.md**：添加新页面条目到相应分类。

**Log.md**：

```markdown
## [YYYY-MM-DD] ingest | 标题

- **来源**: `ObsidianRaw/03_Resources/相对路径.md`
- **影响**:
  - **Sources**: 页面名.md
  - **Concepts**: 概念A.md, 概念B.md
  - **Entities**: 实体A.md
  - **Outputs**: 输出A.md（如有）
  - **Index.md**, **Log.md**
- **备注**: 核心发现摘要，以及本次摄取对现有理解造成的变化
```

## 跳过规则

- 文件已在 Wiki/Sources 中有记录且未变更
- 用户明确指定的排除目录

## 批量处理规则

- 文件数 > 10 时分批处理，每批最多 5 个文件
- 每批完成后报告进度：

```markdown
处理进度：5/N
- ✅ `Academic/File1.md` → [[Sources/File1]]
- ⏭️ `Personal/Sensitive.md` - 跳过（敏感内容）
```

## 完成验证

```
- [ ] Sources 页面已创建
- [ ] Frontmatter 字段完整（见 frontmatter_template.md）
- [ ] 受影响的稳定知识页：已更新"当前综合"部分或标注矛盾
- [ ] 相关分析输出页：已标记为待重审或已更新
- [ ] Index.md 已更新
- [ ] Log.md 已记录，包含"对现有理解造成的变化"
```

## 完成后

执行 `qmd update && qmd embed` 更新搜索索引。若 `qmd` 不可用，跳过索引刷新，在报告中标记"索引待更新"。

## 与 syncing-wiki 的关系

本 Skill 可被 `syncing-wiki` 调用作为阶段 3（执行处理）。独立使用时，完成后必须验证、更新 Log.md、输出处理报告。

## 错误处理

| 错误类型 | 处理方式 |
|---------|---------|
| 源文件读取失败 | 记录失败原因，跳过该文件并继续 |
| frontmatter 字段缺失 | 按模板自动补齐；关键字段缺失时标注"需人工复核" |
| 概念消歧结果不确定 | 不执行强制合并，创建新页面并在 `disambiguation` 中记录原因 |
| `Index.md` 更新冲突 | 保留本次变更，输出"索引冲突待人工合并" |
