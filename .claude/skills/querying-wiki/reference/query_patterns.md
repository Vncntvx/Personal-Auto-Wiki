# 查询模式参考

## 内容目录

- 常用查询模式
- query_hash 计算
- 查询优化技巧
- 输出格式

## 术语定义

术语定义见 AGENTS.md。

## 常用查询模式

### 概念定义查询

查找某个概念的定义和解释：

```bash
qmd query $'intent: 概念定义\nlex: 概念名 定义\nvec: 什么是概念名，概念名的含义和解释' -n 3
```

### 对比查询

查找多个概念的对比：

```bash
qmd query $'intent: 概念对比\nlex: 概念A 概念B 对比 区别\nvec: 概念A和概念B的区别与联系' -n 5
```

### 方法论查询

查找某类方法或技术：

```bash
qmd query $'intent: 方法论\nlex: 方法关键词\nvec: 如何实现某个目标的方法和技术' -n 5
```

### 人物/实体查询

查找人物或组织信息：

```bash
qmd query $'intent: 人物信息\nlex: 人物名' -n 3
```

### 时间线查询

查找发展历程或演变：

```bash
qmd query $'intent: 发展历程\nlex: 主题 发展 阶段 演变\nvec: 某主题的发展历程和演变阶段' -n 5
```

### 知识缺口识别查询

当怀疑 Wiki 缺少能直接回答某主题的稳定知识页或分析输出页时，先检索 Concepts/Entities/Outputs，再判断是否需要从多个 Sources 拼接答案：

```bash
# 示例：检索某主题是否已有相关稳定知识页或分析输出页
qmd query $'intent: 稳定知识页检索\nlex: 韧性 综述 对比\nvec: 韧性主题的分析与比较框架' -n 5
```

**注意**：将 `韧性` 替换为实际主题词。

### 输出晋升查询

判断某个查询结果是否值得沉淀为 `Outputs/`，检索是否存在类似的可复用输出：

```bash
# 示例：检索类似主题的已有输出
qmd query $'intent: 输出复用检索\nlex: 韧性 比较 分析\nvec: 韧性主题的对比分析与结论总结' -n 5
```

**注意**：将主题词替换为实际查询主题，判断是否已有类似输出，避免重复沉淀。

### Outputs 去重检测

保存到 `Outputs/` 前，检测是否存在相似问题：

```bash
# 检测相似输出（相似度阈值 0.75）
qmd query $'intent: 相似输出检测\nvec: 韧性理论的发展阶段有哪些' -c wiki -n 5 --min-score 0.75
```

**检测流程**：
1. 先计算 query_hash（问题标准化后的 hash）
2. 检查是否有完全相同的 query_hash
3. 如无，执行语义相似性搜索（阈值 0.75）
4. 相似度 ≥ 0.75 时提示用户选择：合并/替代/独立创建

> 建议：在检索前先读取 `Wiki/Index.md`，再执行相似输出检测。

## query_hash 计算

用于 Outputs 去重和相似问题检测。将问题标准化后计算 SHA256 前 8 位。

**标准化规则**：小写 + 去标点 + 合并空白 + 去首尾空白

```bash
echo -n "韧性理论的发展阶段有哪些" | tr '[:upper:]' '[:lower:]' | sed 's/[[:punct:]]/ /g; s/[[:space:]]\+/ /g; s/^ //; s/ $//' | openssl dgst -sha256 | sed 's/.*= //' | cut -c1-8
```

## 查询优化技巧

### 使用 intent 消歧义

intent 帮助消歧义，提高相关性：

```bash
# 不好：太模糊
qmd vsearch "韧性"

# 好：明确意图
qmd query $'intent: 韧性理论演变历程\nvec: 韧性理论的发展阶段'
```

### 混合搜索

lex 捕捉精确术语，vec 捕捉语义关联：

```bash
qmd query $'lex: 贝叶斯 网络\nvec: 贝叶斯网络的应用和方法'
```

### 调整分数阈值

过滤低相关性结果：

```bash
qmd query "查询" --min-score 0.6
```

### 性能调优

降低重排序候选数可提速：

```bash
qmd query "查询" -C 20   # 默认 40，降低可提速
qmd query "查询" --no-rerank  # 跳过 LLM 重排序，仅用 RRF 分数（CPU 最快）
```

## 输出格式

```bash
# 仅返回文件路径
qmd query "查询" --files

# JSON 格式（适合程序处理）
qmd query "查询" --json

# 完整文档（非片段）
qmd query "查询" --full

# 显示分数追踪（调试用）
qmd query "查询" --explain
```
