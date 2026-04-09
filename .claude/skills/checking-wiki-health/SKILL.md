---
name: checking-wiki-health
description: 检测 PersonalWiki 知识库的健康状态，包括孤立页面、矛盾标注、概念缺口、未同步文件。当用户请求"健康检查"、"检查Wiki状态"、"诊断"时使用。
---

# Wiki 健康检查

定期或按需检查 Wiki 知识库的完整性和一致性。

## 参考资料

- 检查清单与报告模板：参阅 [health_checklist.md](reference/health_checklist.md)

## 术语定义

术语定义见 AGENTS.md。

## 触发条件

- 用户请求"健康检查"、"检查 Wiki 状态"、"诊断"
- 摄取操作后验证结果
- 定期维护

## 检查项目

### 1. 孤立页面检测（P3）

无入链的页面。如有价值则添加引用，如无价值则考虑删除或合并。

### 2. 未索引页面检测（P2）

存在于文件系统但未在 `Wiki/Index.md` 中列出的页面。更新 Index.md 添加缺失条目。

### 3. 概念缺口检测（P1）

在页面中引用但目标页面不存在。扫描所有 `[[Concepts/xxx]]`、`[[Entities/xxx]]` 链接，检查目标文件是否存在。创建缺失页面。

### 4. 重复概念检测（P1）

可能为同一概念的不同表达。使用 qmd 搜索语义相似的概念页面，检查 `canonical: false` 的别名页面是否正确指向规范条目。

```bash
qmd query $'intent: 概念消歧\nvec: 概念定义' \
  -c wiki --glob "Concepts/**" -n 10 --min-score 0.7
```

确认同一概念则合并并添加 aliases；需消歧则保留独立页面。

### 5. 矛盾标注检测（P1）

搜索 `> [!warning] 矛盾标注` 的页面。人工审核矛盾内容，判断是否需重写相关稳定知识页的"当前综合"部分。

### 6. 未同步文件检测（P0）

调用 `detecting-resources-sync` Skill。识别新增、变更、删除、重命名文件。

### 7. 认知质量检测（P0，量化）

结构上完整但知识仍然浅层、过时或未沉淀的页面。量化指标与质量评分公式见 [health_checklist.md](reference/health_checklist.md)。

**检测方法**：读取页面 frontmatter 中的 `stats` 字段；如无则实时计算并填充。识别低质量页面和来源变更（modified）但未重审的页面。

**处理建议**：
- 低质量页面：标注「待改进」，建议补充来源或扩展定义
- 来源变更（modified）：标注「待重审」，检查相关稳定知识页是否需更新

### 8. Schema 时效检测（P3）

检查 VERSION 和 CHANGELOG.md 是否与当前 Schema 实际状态一致：
- AGENTS.md 的 updated 字段与最近 CHANGELOG 条目是否匹配
- 是否存在未记录在 CHANGELOG 中的 Schema 变更
- VERSION 是否反映当前变更级别

## 报告格式

参阅 [health_checklist.md](reference/health_checklist.md) 中的报告模板。

## 日志要求

健康检查结束后，若识别到以下任一情况，必须写入 `Wiki/Log.md`：
- 低质量页面清单
- 来源变更导致待重审
- 明确的知识缺口或矛盾重审建议

日志类型使用：`lint`

## 相关 Skill

- 调用：`detecting-resources-sync`（检测未同步文件）
- 后续：`ingesting-resources`（处理未同步文件）

## 错误处理

| 错误类型 | 处理方式 |
|---------|---------|
| `qmd` 不可用 | 退化到 `Index.md` + 文件直读；报告标注"语义检测已降级" |
| frontmatter 缺失统计字段 | 实时计算并补齐 `stats`，无法计算时标记"待补数据" |
| 链接解析失败 | 报告中输出原始链接文本并标注"需人工确认" |
