---
name: ingesting-resources
description: 处理 ObsidianRaw/03_Resources 目录中的 Markdown 文件，生成摘要页面、提取概念实体、更新索引。当用户请求"摄取"、"处理新文件"、"更新Wiki"、"导入"时使用。
---

# 资源摄取

将 Resources 目录的原始文件转化为 Wiki 知识库页面。

## 触发条件

- 用户明确请求摄取特定文件或目录
- `detecting-resources-sync` 检测到新增/变更文件后，用户确认摄取
- 用户请求"处理新文件"、"更新 Wiki"

## 范围限制

**仅处理** `ObsidianRaw/03_Resources/` 目录。

其他 PARA 目录不处理：
- `00_Inbox/` — 临时收件箱
- `01_Projects/` — 当前项目，行动导向
- `02_Areas/` — 持续责任区域
- `04_Archive/` — 已完成内容

## 摄取流程

### 步骤 1：读取源文件

读取指定文件的完整内容，识别结构：
- 标题层级
- 章节划分
- 列表和表格
- 代码块

### 步骤 2：生成 Sources 页面

在 `Wiki/Sources/` 创建摘要页面。

**Frontmatter 格式**：

```yaml
---
title: 页面标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: source
tags: [相关标签]
source_path: Resources相对路径.md
source_hash: SHA256前8位
source_mtime: ISO8601时间戳
---
```

**内容结构**：

```markdown
# 标题

> 来源：`ObsidianRaw/03_Resources/相对路径.md`

## 摘要

[1-3 句话概括核心内容]

## 核心内容

[按原文结构提炼]

## 关键概念

- [[Concepts/概念A]]
- [[Concepts/概念B]]

## 相关实体

- [[Entities/实体A]]
```

### 步骤 3：提取概念和实体

**概念提取标准**：
- 方法论、理论框架、术语定义
- 可复用的抽象知识
- 命名：`Wiki/Concepts/{概念名}.md`

**实体提取标准**：
- 人物、组织、工具、数据库
- 具体的可命名对象
- 命名：`Wiki/Entities/{实体名}.md`

### 步骤 4：关联影响分析

对每个提取的概念/实体：

**已存在**：
- 添加新引用来源
- 更新定义（如有补充）
- 标注矛盾（如有冲突）

**不存在**：
- 创建新页面

**矛盾标注格式**：

```markdown
> [!warning] 矛盾标注
> 本页面与 [[Sources/另一来源]] 存在矛盾：
> - 本页面观点：...
> - 另一来源观点：...
```

### 步骤 5：更新索引和日志

**Index.md**：添加新页面条目到相应分类

**Log.md**：

```markdown
## [YYYY-MM-DD] ingest | 标题

- **来源**: `ObsidianRaw/03_Resources/相对路径.md`
- **影响**: 
  - **Sources**: 页面名.md
  - **Concepts**: 概念A.md, 概念B.md
  - **Entities**: 实体A.md
  - **Index.md**, **Log.md**
- **备注**: 核心发现摘要
```

## 跳过规则

以下情况跳过摄取：
- 文件已在 Wiki/Sources 中有记录且未变更
- 用户明确指定的排除目录

## 完成后

执行 `qmd update && qmd embed` 更新搜索索引。
