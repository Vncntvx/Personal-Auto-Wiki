# Changelog

Schema 变更记录。本文件追踪 AGENTS.md、Skills、Reference 及 VERSION 的所有变更。

## 版本规则

- **发布版本**（`vYYYY.MM`）：架构性变更、术语表扩展、新增 Skill 等显著变化
- **补丁版本**（`vYYYY.MM.N`）：模板微调、措辞修正、Reference 内容迁移等局部变更
- 补丁累计到一定量或发生架构性变更时，升级到新的发布版本

## 维护规则

- 每次 Schema 变更必须追加条目
- VERSION 变更时创建新版本头；同一版本内的增量变更追加到当前版本头下
- Rationale 段记录"为什么改"，支持后续回溯决策依据

---

## [v2026.04.1] — 2026-04-09

### Added

- `CHANGELOG.md`：Schema 变更记录文件，与 `Wiki/Log.md`（知识操作）职责分离
- AGENTS.md §3 第 5 条共演原则：Schema 与知识库共演，变更必须记录到 CHANGELOG.md
- AGENTS.md §6 Schema 变更指向：明确 Schema 变更不在 Log.md 记录
- AGENTS.md §10 CHANGELOG 引用
- checking-wiki-health 第 8 项 Schema 时效检测（P3）
- health_checklist.md 第 9 项检查流程 + 报告模板 + 优先级表条目
- VERSION 补丁版本格式（`vYYYY.MM.N`）

### Changed

- 移除 AGENTS.md §10 中 `Python 命令示例统一使用 python3`（改用跨平台 CLI）
- query_patterns.md 中 `python3` heredoc 替换为 `echo | tr | sed | openssl` 管道
- README_CN.md / README.md 更新：新增 syncing-wiki skill、VERSION 文件、术语对齐

### Rationale

- 实现 llm-wiki.md 的 Schema 共演理念：用户与 LLM 共同迭代改进 Schema
- 补齐 Schema 变更追踪缺口：此前 AGENTS.md/Skills/Reference 的修改无记录
- 统一使用跨平台 CLI，移除 Python3 依赖

---

## [v2026.04] — 2026-04-09

### Changed

- AGENTS.md 大幅重构（462 → 197 行），符合 AGENTS.md BestPractices 200 行目标
- 全局术语表扩展（5 → 11 项），统一跨文档术语
- 知识演化原则精简为四条，质量评分公式迁移至 health_checklist.md
- 协作规范精简：frontmatter 模板、Hash 计算命令迁移至 reference 文件
- 日志与索引压缩：删除完整模板，保留可解析约束
- 搜索优先级与响应风格压缩

### Added

- `VERSION` 文件：集中管理架构规范版本
- `disambiguation_guide.md`：消歧流程从 ingesting-resources/SKILL.md 提取为独立 reference
- `health_checklist.md`：增补质量评分公式 + 评分标准 + 质量判定标签（从 AGENTS.md 和 SKILL.md 迁入）
- `frontmatter_template.md`：增补 Hash 计算段（从 AGENTS.md 和 detecting-resources-sync/SKILL.md 迁入）
- `query_patterns.md`：增补 query_hash 计算和查询优化技巧（从 querying-wiki/SKILL.md 迁入）

### Removed

- AGENTS.md 中冗余的质量评分公式、frontmatter 模板、Hash 命令、qmd 命令列表、Index.md 模板、工作流示例
- 各 SKILL.md 中的"术语对齐"段（统一指向 AGENTS.md）
- `page_templates.md` 中与 frontmatter_template.md 重复的"Frontmatter 字段参考"段

### Rationale

- 消除跨文档冗余（质量公式出现 3 处、frontmatter 模板出现 4+ 处、Hash 命令出现 3 处）
- 确立"每项知识只有一个规范位置"的内容归属原则
- 遵循 Claude Skills BestPractices 和 AGENTS.md BestPractices 的行数与结构约束
