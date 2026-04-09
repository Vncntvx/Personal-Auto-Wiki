# 查询模式参考

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
