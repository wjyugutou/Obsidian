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

### 3. Stream ↔ Iterator 互转

#### Iterator → Stream

```java
Iterator<String> it = list.iterator();
// Java 8 没有直接 API，需借助 Spliterator
Stream<String> stream = StreamSupport.stream(
    Spliterators.spliteratorUnknownSize(it, Spliterator.ORDERED),
    false  // parallel = false
);
```

#### Stream → Iterator
```java
Iterator<String> it = stream.iterator();
// 注意：这会触发流的终端操作，流被消费后不可再用
```

### 4. 如何选择？决策指南

|场景|推荐|原因|
|:--|:--|:--|
|简单遍历 + 无副作用|Stream|声明式，可读性强|
|遍历时需要删除/修改元素|Iterator|Stream 不支持安全修改源集合|
|需要索引/下标|for-i / ListIterator|Stream 天然无索引概念|
|大数据集 + 多核环境|parallelStream|自动并行分片|
|复杂状态依赖的遍历|Iterator / for|Stream 应避免有状态 lambda|
|仅需判断存在/计数|Stream|`anyMatch/count` 可短路优化|
|性能极度敏感的热路径|传统 for|Stream 有对象创建开销|
