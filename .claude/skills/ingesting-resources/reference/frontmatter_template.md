# Sources 页面 Frontmatter 模板

## 内容目录

- 标准模板
- 字段说明
- 追踪字段用途
- Hash 计算
- docid 生成命令
- 示例

## 术语定义

术语定义见 AGENTS.md。

## 标准模板

```yaml
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
aliases:                          # 历史路径（可选）
  - 旧路径/旧文件名.md
---
```

## 字段说明

| 字段 | 必需 | 类型 | 说明 |
|------|------|------|------|
| `title` | 是 | string | 页面标题，通常与源文件标题一致 |
| `created` | 是 | date | 创建日期，格式 YYYY-MM-DD |
| `updated` | 是 | date | 最后更新日期 |
| `type` | 是 | string | 固定值 `source` |
| `tags` | 是 | array | 标签数组，用于分类 |
| `docid` | 是 | string | 全局唯一标识符，格式：`{语义前缀}_{UUID前8位}` |
| `source_path` | 是 | string | 相对于 `ObsidianRaw/03_Resources/` 的路径 |
| `source_hash` | 是 | string | 文件内容 SHA256 前 8 位 |
| `source_mtime` | 是 | string | 文件最后修改时间，ISO 8601 格式 |
| `aliases` | 否 | array | 历史路径列表，用于追溯文件重命名历史 |

## 追踪字段用途

### docid（全局唯一标识符）

- **用途**：唯一标识一个来源，即使文件重命名、移动也能追踪
- **生成规则**：
  - 语义前缀：从标题提取，中文转拼音、英文保留，取前 20 字符
  - UUID 部分：首次创建时随机生成 8 位十六进制
- **示例**：`韧性` → `renxing_a1b2c3d4`，`Bayesian Network` → `bayesian_network_e5f6a7b8`
- **重要**：docid 一旦生成，永久不变

### source_path

- 定位原始文件
- 用于检测文件是否被移动或删除
- 格式：`Academic/Research Notes/Theory/名词概念/韧性.md`

### source_hash

- 检测文件内容是否变更
- 计算 SHA256 后取前 8 位
- 对比时区分大小写

### source_mtime

- 快速筛选变更文件
- 避免每次都计算哈希
- 格式：`2026-04-08T17:30:00Z`

### aliases（历史路径）

- 记录文件的历史路径
- 当文件重命名时，旧路径会自动添加到此字段
- 用于追溯来源演变历史

## Hash 计算

使用 SHA256 前 8 位十六进制字符检测内容变更。

**跨平台统一命令（推荐）**：

```bash
openssl dgst -sha256 "$file" | sed 's/.*= //' | cut -c1-8
```

**替代命令**：

| 平台 | 命令 |
|------|------|
| macOS / Linux | `sha256sum "$file" \| cut -c1-8` |
| Windows (PowerShell) | `(Get-FileHash "$file" -Algorithm SHA256).Hash.Substring(0,8)` |

## docid 生成命令

**UUID 前 8 位生成（统一使用 openssl）**：

```bash
# 跨平台
openssl rand -hex 4
```

**语义前缀生成**：

由 LLM 在创建页面时生成，规则如下：

- 中文标题：转换为拼音，取前 20 字符
- 英文标题：小写，空格替换为下划线，取前 20 字符
- 示例：`韧性` → `renxing`，`Bayesian Network` → `bayesian_network`

**完整 docid 示例**：

```
renxing_a1b2c3d4
bayesian_network_e5f6a7b8
```

## 示例

```yaml
---
title: 韧性
created: 2026-04-08
updated: 2026-04-09
type: source
tags: [系统理论, 生态学, 韧性研究]
docid: renxing_a1b2c3d4
source_path: Academic/韧性理论/韧性概念.md
source_hash: 45ea3a15
source_mtime: 2026-04-09T10:00:00Z
aliases:
  - Academic/Research Notes/Theory/名词概念/韧性.md
---
```
