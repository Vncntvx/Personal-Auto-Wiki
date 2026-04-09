# 同步工作流示例

## 内容目录

- 示例 1：少量新文件
- 示例 2：批量处理（分批）
- 示例 3：有跳过和错误

## 术语约定

- 同步状态统一使用：新增 (`new`) / 变更 (`modified`) / 删除 (`deleted`) / 同步 (`synced`)。
- 重命名 (`renamed`) 为推断状态。

## 示例 1：少量新文件

**用户输入**：`同步并处理`

**阶段 1 输出**：

```markdown
## Resources 同步状态（四态）

| 类型 | 数量 |
|------|------|
| 新增文件 (`new`) | 3 |
| 变更文件 (`modified`) | 0 |
| 删除文件 (`deleted`) | 0 |
| 同步文件 (`synced`) | 45 |

### 新增文件列表
1. `Academic/Research Notes/Methods/NewMethod.md`
2. `Academic/Tool/NewTool.md`
3. `Writing/写作思路/NewIdea.md`
```

**阶段 2 输出**：

```markdown
## 待处理文件

### 新增文件 (3)
1. `Academic/Research Notes/Methods/NewMethod.md`
2. `Academic/Tool/NewTool.md`
3. `Writing/写作思路/NewIdea.md`

---
**是否继续处理？** 输入"是"继续。
```

**阶段 3 输出**：

```markdown
处理进度：3/3
- ✅ `Academic/Research Notes/Methods/NewMethod.md` → [[Sources/NewMethod]]
- ✅ `Academic/Tool/NewTool.md` → [[Sources/NewTool]]
- ✅ `Writing/写作思路/NewIdea.md` → [[Sources/NewIdea]]

综合更新：
- ✅ [[Concepts/NewMethod]] 已补充当前综合
- ✅ [[Entities/NewTool]] 已更新说明
- ✅ [[Outputs/2026-04-08-new-method-comparison]] 已检查为无需更新
```

**阶段 5 Log.md 记录**：

```markdown
## [2026-04-08] sync | Wiki 同步处理

- **来源**: `ObsidianRaw/03_Resources/`
- **影响**:
  - **新增**: NewMethod.md, NewTool.md, NewIdea.md
  - **Sources**: +3 个页面
  - **Concepts**: +2 个页面
  - **Outputs**: 更新 1 个页面
  - **Index.md**, **Log.md**
- **备注**: 处理 3 个新增文件；新增方法页更新了现有综合理解
```

**阶段 6 完成报告**：

```markdown
# 同步处理完成报告

**时间**：2026-04-08 15:30

## 处理统计

| 类型 | 数量 | 状态 |
|------|------|------|
| 新增文件 (`new`) | 3 | ✅ 完成 |
| 变更文件 (`modified`) | 0 | - |
| 删除文件 (`deleted`) | 0 | - |

## 验证结果
- ✅ Sources 页面：全部创建
- ✅ Frontmatter：全部完整
- ✅ Concepts / Entities：已复核
- ✅ Outputs：已检查
- ✅ 涉及变化的 Concepts/Entities：已更新"当前综合"部分
- ✅ Index.md：已更新
- ✅ Log.md：已记录
- ✅ qmd 索引：已更新
```

---

## 示例 2：批量处理（分批）

**用户输入**：`同步并处理`

**阶段 1 输出**：

```markdown
## Resources 同步状态（四态）

| 类型 | 数量 |
|------|------|
| 新增文件 (`new`) | 15 |
| 变更文件 (`modified`) | 3 |
| 删除文件 (`deleted`) | 1 |
| 同步文件 (`synced`) | 45 |

### 重命名文件
- `Academic/OldName.md` → `Academic/NewName.md`（hash 匹配）
```

**阶段 3 输出**（分批处理）：

```markdown
处理进度：5/18
- ✅ `Academic/File1.md` → [[Sources/File1]]
- ✅ `Academic/File2.md` → [[Sources/File2]]
- ✅ `Academic/File3.md` → [[Sources/File3]]
- ✅ `Academic/File4.md` → [[Sources/File4]]
- ✅ `Academic/File5.md` → [[Sources/File5]]

综合更新：
- ✅ [[Concepts/TopicA]] 已重写当前综合
- ✅ [[Entities/ToolB]] 已补充说明
- ⚠️ [[Outputs/2026-04-01-topic-a-review]] 标记为待重审

**继续处理下一批？** 输入"继续"。
```

---

## 示例 3：有跳过和错误

**阶段 3 输出**：

```markdown
处理进度：5/8
- ✅ `Academic/ValidFile.md` → [[Sources/ValidFile]]
- ⏭️ `Writing/Confidential-Draft.md` - 跳过（用户明确排除）
- ✅ `Academic/Another.md` → [[Sources/Another]]
- ❌ `Academic/NotFound.md` - 错误：文件不存在
- ✅ `Academic/LastOne.md` → [[Sources/LastOne]]
```

**阶段 6 完成报告**：

```markdown
# 同步处理完成报告

## 处理统计

| 类型 | 数量 | 状态 |
|------|------|------|
| 新增文件 | 5 | ✅ 完成 |
| 跳过文件 | 1 | ⏭️ 跳过 |
| 错误 | 1 | ❌ 失败 |

### 错误详情
- `Academic/NotFound.md` - 文件不存在

### 跳过详情
- `Writing/Confidential-Draft.md` - 用户明确排除
```
