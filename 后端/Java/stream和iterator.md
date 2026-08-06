### 1. Iterator：传统的命令式遍历

`Iterator` 是 Java 集合框架最基础的遍历接口，属于**外部迭代**。你需要手动控制“有没有下一个”以及“取下一个”。

```java
List<String> list = Arrays.asList("Vue", "React", "Angular", "Svelte");

// ✅ 标准写法（推荐）
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String item = it.next();
    if ("React".equals(item)) {
        it.remove(); // ⚠️ Iterator 唯一安全的删除元素方式
    }
}

// ❌ 错误示范：在 for-each 或 stream 中直接 remove 会抛 ConcurrentModificationException
```

**特点:**

- 命令式风格，代码较冗长
- 支持遍历时安全删除元素
- 只能单次遍历，不可复用
- 串行执行，无法利用多核

### 2. Stream：声明式数据处理管道

`Stream` 是 Java 8+ 引入的抽象，属于**内部迭代**。你只需描述“做什么”，而非“怎么做”。类似前端的链式调用。
```java
List<String> result = list.stream()       // 创建流
    .filter(s -> s.length() > 4)          // 过滤：长度>4
    .map(String::toUpperCase)             // 映射：转大写
    .sorted()                             // 排序
    .collect(Collectors.toList());        // 终端操作：收集结果

// result: ["ANGULAR", "REACT", "SVLTE"] → 实际为 ["ANGULAR", "REACT", "SVLTE"] 修正: SVLTE长度5
```

**三大核心操作类型：**
|操作类型|说明|示例|类比前端|
|:--|:--|:--|:--|
|创建|从集合/数组/生成器创建流|`stream()`, `of()`, `generate()`|`[...arr]`|
|中间操作|返回新 Stream，惰性求值|`filter`, `map`, `flatMap`, `sorted`, `distinct`|`.filter().map()`|
|终端操作|触发执行，消费流|`collect`, `forEach`, `reduce`, `count`, `anyMatch`|`.reduce()`, `.forEach()`|

⚡ **惰性求值**：中间操作不会立即执行，只有遇到终端操作时才一次性流水线处理。这意味着可以优化执行计划（如短路、融合循环）。

