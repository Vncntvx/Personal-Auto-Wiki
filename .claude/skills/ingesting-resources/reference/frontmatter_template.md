# Sources 页面 Frontmatter 模板

## 标准模板

```yaml
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
```

## 字段说明

| 字段 | 必需 | 类型 | 说明 |
|------|------|------|------|
| `title` | 是 | string | 页面标题，通常与源文件标题一致 |
| `created` | 是 | date | 创建日期，格式 YYYY-MM-DD |
| `updated` | 是 | date | 最后更新日期 |
| `type` | 是 | string | 固定值 `source` |
| `tags` | 是 | array | 标签数组，用于分类 |
| `source_path` | 是 | string | Resources 目录中的相对路径 |
| `source_hash` | 是 | string | 文件内容 SHA256 前 8 位 |
| `source_mtime` | 是 | string | 文件最后修改时间，ISO 8601 格式 |

## 追踪字段用途

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

## 示例

```yaml
---
title: 韧性
created: 2026-04-08
updated: 2026-04-08
type: source
tags: [系统理论, 生态学, 韧性研究]
source_path: Academic/Research Notes/Theory/名词概念/韧性.md
source_hash: a1b2c3d4
source_mtime: 2026-04-08T17:30:00Z
---
```
