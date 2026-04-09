---
name: detecting-resources-sync
description: 检测 Resources 与 Wiki/Sources 的同步状态，识别新增、变更、删除并推断重命名。当用户请求"检测新文件"、"同步状态"时使用。
---

# Resources 同步检测

检测 Resources 目录与 Wiki/Sources 页面的同步状态。

## 参考资料

- 报告格式：参阅 [sync_report_format.md](reference/sync_report_format.md)
- Frontmatter 追踪字段：参阅 [frontmatter_template.md](../ingesting-resources/reference/frontmatter_template.md)

## 术语定义

术语定义见 AGENTS.md。

## 触发条件

- 用户请求"检测新文件"、"同步状态"
- 健康检查流程中自动调用
- 摄取前确认待处理文件

## 检测流程

### 步骤 1：扫描 Resources 目录

```
路径：ObsidianRaw/03_Resources/
模式：**/*.md
```

对每个文件记录：相对路径、内容哈希（`openssl dgst -sha256 "$file" | sed 's/.*= //' | cut -c1-8`）、修改时间（ISO 8601）。

### 步骤 2：读取 Sources 索引

遍历 `Wiki/Sources/*.md`，提取 frontmatter 追踪字段（docid、source_path、source_hash、source_mtime、aliases），建立 docid 映射。字段说明参阅 [frontmatter_template.md](../ingesting-resources/reference/frontmatter_template.md)。

### 步骤 3：四态比对

| 状态 | 条件 |
|------|------|
| **新增 (`new`)** | Resources 中存在但 Sources 无记录，且无 hash 匹配 |
| **变更 (`modified`)** | path 相同但 hash 不同 |
| **删除 (`deleted`)** | Sources 有记录但 Resources 文件不存在，且无 hash 匹配 |
| **同步 (`synced`)** | path 和 hash 都相同 |

### 步骤 4：重命名检测算法

当检测到「删除 + 新增」组合时：

```
1. 收集所有「删除」文件的 source_hash
2. 收集所有「新增」文件的 content_hash
3. hash 完全匹配 → 判定为「重命名 (renamed)」
   - 更新 Sources 页面的 source_path，保留 docid
   - 将旧路径添加到 aliases 字段
4. hash 不匹配 → 标记为「疑似重命名」，询问用户确认
```

## 输出格式

参阅 [sync_report_format.md](reference/sync_report_format.md)。

## 后续操作

| 状态 | 建议操作 |
|------|----------|
| 重命名文件 | 自动更新 source_path，保留 docid，添加旧路径到 aliases |
| 新增文件 | 调用 `syncing-wiki` 或 `ingesting-resources` 摄取 |
| 变更文件 | 调用 `syncing-wiki` 或 `ingesting-resources` 更新，并复核受影响的稳定知识页与分析输出页 |
| 删除文件 | 标记 Sources 页面或删除，并检查受影响的稳定知识页与分析输出页 |
| 疑似重命名 | 询问用户确认 |

检测完成后，引导用户选择下一步操作：

```markdown
---
**发现 N 个待处理文件**

是否继续处理？
- 输入"是"或"继续" — 执行完整同步处理
- 输入"仅处理新增" — 仅处理新增文件
- 输入具体文件路径 — 仅处理指定文件
```

## 日志要求

- 本 Skill 单独运行时：默认只输出检测报告，不强制写 `Wiki/Log.md`
- 若作为 `syncing-wiki` 阶段 1 运行：由 `syncing-wiki` 在阶段 5 统一留痕

## 边界情况

| 情况 | 处理方式 |
|------|----------|
| 文件重命名（内容不变） | hash 匹配 → 自动判定为重命名 |
| 文件重命名 + 内容修改 | hash 不匹配 → 标记为「疑似重命名」，询问用户 |
| 批量移动目录 | 多个 hash 匹配 → 批量更新 source_path |
| docid 缺失 | 标记为「需修复」，下次同步时自动生成 |
| source_path 缺失 | 标记为「需修复」，从 docid 或内容推断路径 |

## 与 syncing-wiki 的关系

本 Skill 是 `syncing-wiki` 的阶段 1（检测同步状态）。独立使用时，输出后应询问用户是否继续处理。

## 错误处理

| 错误类型 | 处理方式 |
|---------|---------|
| `Wiki/Sources` 页面 frontmatter 缺失 | 标记为"需修复"，不阻塞其余文件检测 |
| 哈希计算失败 | 回退到 mtime + 文件大小比较，报告中标注"低置信度" |
| 路径含特殊字符 | 使用引号包裹路径并保留原始路径输出 |
