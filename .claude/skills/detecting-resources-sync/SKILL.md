---
name: detecting-resources-sync
description: 检测 Resources 与 Wiki/Sources 的同步状态，识别新增、变更、删除的文件。当用户请求"检测新文件"、"同步状态"时使用。
---

# Resources 同步检测

检测 Resources 目录与 Wiki/Sources 页面的同步状态。

## 触发条件

- 用户请求"检测新文件"、"同步状态"
- 健康检查流程中自动调用
- 摄取前确认待处理文件

## 检测机制

### 三重追踪

Sources 页面 frontmatter 必须包含：

```yaml
source_path: Academic/Research Notes/Theory/名词概念/韧性.md
source_hash: a1b2c3d4
source_mtime: 2026-04-08T17:30:00Z
```

| 字段 | 用途 |
|------|------|
| `source_path` | 定位原始文件 |
| `source_hash` | 检测内容变更（SHA256 前 8 位） |
| `source_mtime` | 快速筛选，避免全量计算哈希 |

## 检测流程

### 步骤 1：扫描 Resources 目录

```
路径：ObsidianRaw/03_Resources/
模式：**/*.md
```

对每个文件记录：
- 相对路径（相对于 Resources/）
- 内容哈希（SHA256 前 8 位）
- 修改时间（ISO 8601）

### 步骤 2：读取 Sources 索引

遍历 `Wiki/Sources/*.md`，提取 frontmatter 中的追踪字段。

### 步骤 3：三路比对

| 状态 | 条件 |
|------|------|
| **新增** | Resources 中存在但 Sources 无记录 |
| **变更** | path 相同但 hash 不同 |
| **删除** | Sources 有记录但 Resources 文件不存在 |
| **同步** | path 和 hash 都相同 |

## 输出格式

```markdown
## Resources 同步状态 [YYYY-MM-DD HH:MM]

### 新增文件 (N)
| 路径 | 修改时间 |
|------|----------|
| `Academic/NewTopic.md` | 2026-04-08 09:00 |

### 已变更 (N)
| 路径 | 旧哈希 | 新哈希 |
|------|--------|--------|
| `Academic/Theory/韧性.md` | a1b2c3d4 | e5f6g7h8 |

### 原始文件已删除 (N)
| 路径 | Sources 页面 |
|------|--------------|
| `Academic/OldTopic.md` | [[Sources/OldTopic]] |

### 已同步 (N)
- 共 N 个文件处于同步状态
```

## 后续操作

| 状态 | 建议操作 |
|------|----------|
| 新增文件 | 调用 `syncing-wiki` 或 `ingesting-resources` 摄取 |
| 变更文件 | 调用 `syncing-wiki` 或 `ingesting-resources` 更新 |
| 删除文件 | 标记 Sources 页面或删除 |

## 边界情况

| 情况 | 处理方式 |
|------|----------|
| 文件重命名 | 识别为「删除 + 新增」，需用户确认 |
| source_path 缺失 | 标记为「需修复」，从内容推断路径 |
| 特殊字符路径 | 使用引号包裹，确保路径正确 |

## Hash 计算

使用系统命令计算 SHA256 前 8 位：

### macOS / Linux

```bash
sha256sum "$file" | cut -c1-8
# 或
openssl dgst -sha256 "$file" | awk '{print $2}' | cut -c1-8
```

### Windows (PowerShell)

```powershell
(Get-FileHash "$file" -Algorithm SHA256).Hash.Substring(0,8)
```

### 跨平台统一命令（推荐）

```bash
openssl dgst -sha256 "$file" | sed 's/.*= //' | cut -c1-8
```

**注意**：哈希值取 SHA256 输出的前 8 位十六进制字符。

---

## 后续操作引导

检测完成后，应引导用户进行下一步操作：

### 输出后询问

```markdown
---
**发现 N 个待处理文件**

是否继续处理？
- 输入"是"或"继续" — 执行完整同步处理
- 输入"仅处理新增" — 仅处理新增文件
- 输入具体文件路径 — 仅处理指定文件
```

### 推荐操作

| 检测结果 | 推荐操作 |
|---------|---------|
| 有新增文件 | 调用 `sync-and-process` 完整处理 |
| 有变更文件 | 调用 `ingesting-resources` 更新 |
| 有删除文件 | 标记对应 Sources 页面 |
| 无变化 | 无需操作 |

---

## 与 syncing-wiki 的关系

本 Skill 是 `syncing-wiki` 的阶段 1（检测同步状态）。

独立使用时，输出后应询问用户是否继续处理。
