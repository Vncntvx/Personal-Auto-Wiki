# 查询模式参考

## 常用查询模式

### 概念定义查询

查找某个概念的定义和解释：

```javascript
mcp__qmd__query({
  searches: [
    { type: "lex", query: "概念名 定义" },
    { type: "vec", query: "什么是概念名，概念名的含义和解释" }
  ],
  intent: "概念定义",
  limit: 3
})
```

### 对比查询

查找多个概念的对比：

```javascript
mcp__qmd__query({
  searches: [
    { type: "lex", query: "概念A 概念B 对比 区别" },
    { type: "vec", query: "概念A和概念B的区别与联系" }
  ],
  intent: "概念对比",
  limit: 5
})
```

### 方法论查询

查找某类方法或技术：

```javascript
mcp__qmd__query({
  searches: [
    { type: "lex", query: "方法关键词" },
    { type: "vec", query: "如何实现某个目标的方法和技术" }
  ],
  intent: "方法论",
  limit: 5
})
```

### 人物/实体查询

查找人物或组织信息：

```javascript
mcp__qmd__query({
  searches: [
    { type: "lex", query: "人物名" }
  ],
  intent: "人物信息",
  limit: 3
})
```

### 时间线查询

查找发展历程或演变：

```javascript
mcp__qmd__query({
  searches: [
    { type: "lex", query: "主题 发展 阶段 演变" },
    { type: "vec", query: "某主题的发展历程和演变阶段" }
  ],
  intent: "发展历程",
  limit: 5
})
```

## 查询优化技巧

### 使用 intent 参数

intent 帮助消歧义，提高相关性：

```javascript
// 不好
mcp__qmd__query({
  searches: [{ type: "vec", query: "韧性" }],
  limit: 5
})

// 好
mcp__qmd__query({
  searches: [{ type: "vec", query: "韧性理论的发展阶段" }],
  intent: "韧性理论演变历程",
  limit: 5
})
```

### 混合搜索

lex 捕捉精确术语，vec 捕捉语义关联：

```javascript
// 推荐
mcp__qmd__query({
  searches: [
    { type: "lex", query: "贝叶斯 网络" },
    { type: "vec", query: "贝叶斯网络的应用和方法" }
  ],
  intent: "贝叶斯网络方法",
  limit: 5
})
```

### 设置 minScore

过滤低相关性结果：

```javascript
mcp__qmd__query({
  searches: [...],
  minScore: 0.6,  // 提高阈值
  limit: 5
})
```
