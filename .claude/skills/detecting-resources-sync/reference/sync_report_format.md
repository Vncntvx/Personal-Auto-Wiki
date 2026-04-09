# 同步报告格式

## 内容目录

- 标准输出格式
- JSON 格式
- 状态定义
- 特殊情况标注

## 术语约定

- 同步状态统一使用：新增 (`new`) / 变更 (`modified`) / 删除 (`deleted`) / 同步 (`synced`)。
- 重命名 (`renamed`) 为推断状态，由 `deleted + new` 的 hash 匹配得到。

## 标准输出格式

```markdown
## Resources 同步状态 [YYYY-MM-DD HH:MM]

### 新增文件 (N)

| 路径 | 修改时间 | 大小 |
|------|----------|------|
| `Academic/NewTopic.md` | 2026-04-08 09:00 | 2.5KB |

### 变更文件 (`modified`) (N)

| 路径 | 旧哈希 | 新哈希 | 修改时间 |
|------|--------|--------|----------|
| `Academic/Theory/韧性.md` | a1b2c3d4 | e5f6a7b8 | 2026-04-08 08:30 |

### 重命名文件 (N)

| 旧路径 | 新路径 | docid |
|--------|--------|-------|
| `Academic/OldName.md` | `Academic/NewName.md` | renxing_a1b2c3d4 |

### 原始文件已删除 (N)

| 路径 | Sources 页面 |
|------|--------------|
| `Academic/OldTopic.md` | [[Sources/OldTopic]] |

### 同步文件 (`synced`) (N)

共 N 个文件处于同步状态。

---

**统计**：
- Resources 文件总数：N
- Sources 页面总数：N
- 待处理：N（新增 N + 变更 N）
```

## JSON 格式（供程序处理）

```json
{
  "scan_time": "2026-04-08T10:30:00Z",
  "resources_total": 64,
  "sources_total": 45,
  "new_files": [
    {
      "path": "Academic/NewTopic.md",
      "mtime": "2026-04-08T09:00:00Z",
      "size": 2560
    }
  ],
  "modified_files": [
    {
      "path": "Academic/Theory/韧性.md",
      "old_hash": "a1b2c3d4",
      "new_hash": "e5f6a7b8",
      "mtime": "2026-04-08T08:30:00Z"
    }
  ],
  "renamed_files": [
    {
      "old_path": "Academic/OldName.md",
      "new_path": "Academic/NewName.md",
      "docid": "renxing_a1b2c3d4"
    }
  ],
  "deleted_files": [
    {
      "path": "Academic/OldTopic.md",
      "source_page": "Wiki/Sources/OldTopic.md"
    }
  ],
  "synced": 43
}
```

## 状态定义

| 状态 | 条件 | 后续操作 |
|------|------|----------|
| new | Resources 有，Sources 无 | 摄取 |
| modified | path 相同，hash 不同 | 重新摄取 |
| deleted | Sources 有，Resources 无 | 标记或删除 |
| synced | path 和 hash 都相同 | 无需操作 |
| renamed | 由 deleted + new 的 hash 匹配推断 | 更新 source_path + aliases |

## 特殊情况标注

```markdown
### 需要修复 (N)

| 路径 | 问题 | Sources 页面 |
|------|------|--------------|
| `Academic/Unknown.md` | source_path 缺失 | [[Sources/Unknown]] |
```
