---
name: syncing-wiki
description: 执行完整的 Wiki 同步工作流（检测→确认→处理→验证→记录→报告）。当用户请求"同步并处理"、"完整同步"、"检测并处理"时使用。
---

# Wiki 同步工作流

完整的 Wiki 同步工作流，确保每一步都正确执行并有验证。

## 参考资料

- 工作流示例：参阅 [workflow-example.md](reference/workflow-example.md)
- 报告格式：参阅 [sync_report_format.md](../detecting-resources-sync/reference/sync_report_format.md)

## 术语定义

术语定义见 AGENTS.md。依赖与回退策略见 AGENTS.md。

## 触发条件

- 用户请求"同步并处理"、"完整同步"、"检测并处理"

## 工作流清单

复制此清单并跟踪进度：

```
同步进度：
- [ ] 阶段 1：检测同步状态
- [ ] 阶段 2：确认处理范围
- [ ] 阶段 3：执行处理
- [ ] 阶段 4：验证结果
- [ ] 阶段 5：记录日志
- [ ] 阶段 6：输出报告
```

## 阶段 1：检测同步状态

1. 调用 `detecting-resources-sync`
2. 输出四态结果：新增 / 变更 / 删除 / 同步，及重命名推断

| 类型 | 数量 |
|------|------|
| 新增文件 (`new`) | N |
| 变更文件 (`modified`) | N |
| 删除文件 (`deleted`) | N |
| 重命名文件 (`renamed`) | N |
| 同步文件 (`synced`) | N |

## 阶段 2：确认处理范围

1. 展示待处理文件列表
2. 标注跳过项（非 Markdown、超出 `ObsidianRaw/03_Resources/` 边界、或用户明确排除）
3. 询问用户是否继续

## 阶段 3：执行处理

按 `ingesting-resources` skill 执行摄取，遵循 AGENTS.md 知识演化原则传播增量。

**批量处理**：文件数 > 10 时分批，每批 5 个，每批完成后报告进度：

```markdown
处理进度：5/20
- ✅ `Academic/NewTopic.md` → [[Sources/NewTopic]]
- ⏭️ `Personal/Sensitive.md` - 跳过
- ❌ `Academic/Error.md` - 错误：文件不存在
```

**验证**：
- [ ] 每个 Sources 页面已创建
- [ ] 每个 frontmatter 字段完整
- [ ] 受影响的稳定知识页与分析输出页已检查
- [ ] Index.md 已更新

## 阶段 4：验证结果

```
- [ ] Sources 页面：N 个新页面已创建
- [ ] Frontmatter：所有字段完整
- [ ] 稳定知识页：受影响页面已复核
- [ ] 分析输出页：相关页面已检查
- [ ] 涉及变化的稳定知识页：已更新"当前综合"部分
- [ ] Index.md：已更新 N 个条目
- [ ] qmd 索引：已更新
```

## 阶段 5：记录日志

**必须执行**。在 `Wiki/Log.md` 追加记录：

```markdown
## [YYYY-MM-DD] sync | Wiki 同步处理

- **来源**: `ObsidianRaw/03_Resources/`
- **影响**:
  - **新增**: 来源A.md, 来源B.md
  - **变更**: 来源C.md, 来源D.md
  - **删除**: 来源E.md（已标记）
  - **跳过**: 敏感文件.md（敏感内容）
  - **Sources**: +N 个页面
  - **Concepts**: +N 个页面
  - **Entities**: +N 个页面
  - **Outputs**: +N / 更新 N 个页面
  - **Index.md**, **Log.md**
- **备注**: 完整同步处理，无错误；并记录本次对现有理解造成的主要变化
```

## 阶段 6：输出报告

```markdown
# 同步处理完成报告

**时间**：YYYY-MM-DD HH:MM

## 处理统计

| 类型 | 数量 | 状态 |
|------|------|------|
| 新增文件 (`new`) | N | ✅ 完成 |
| 变更文件 (`modified`) | N | ✅ 完成 |
| 删除文件 (`deleted`) | N | ⚠️ 已标记 |
| 跳过文件 | N | ⏭️ 跳过 |

## 验证结果
- ✅ Sources 页面：全部创建
- ✅ Frontmatter：全部完整
- ✅ 受影响的稳定知识页：已复核
- ✅ 相关分析输出页：已检查
- ✅ 涉及变化的稳定知识页：已更新"当前综合"部分
- ✅ Index.md：已更新
- ✅ Log.md：已记录
- ✅ qmd 索引：已更新
```

## 错误处理

| 错误类型 | 处理方式 |
|---------|---------|
| `qmd` 不可用 | 跳过索引刷新，输出"待刷新索引"并保留其它阶段结果 |
| frontmatter 不完整 | 创建后验证，补充缺失字段 |

## 执行保证

**必须执行**：阶段 1-6 全部完成；阶段 5 必须写入 Log.md；阶段 6 必须输出完成报告。

**不允许**：❌ 跳过任何阶段 | ❌ 在阶段 5 之前结束 | ❌ 不写入 Log.md 就结束

## 相关 Skill

- 内部调用：`detecting-resources-sync`（阶段 1）
- 内部调用：`ingesting-resources`（阶段 3）
- 后续调用：`checking-wiki-health`（可选验证）
