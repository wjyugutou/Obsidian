## StringBuilder
长文本 大文本 拼接, 性能好, 

|Java StringBuilder|JS/TS 等价思维|说明|
|:--|:--|:--|
|`append(value)`|`arr.push()` / 链式调用|支持所有基本类型、Object、CharSequence|
|`insert(offset, value)`|`str.slice(0,n) + val + str.slice(n)`|指定位置插入|
|`delete(start, end)`|`str.slice()` 重组|删除区间 [start, end)|
|`replace(start, end, str)`|同上|替换区间内容|
|`reverse()`|`[...str].reverse().join('')`|原地反转|
|`toString()`|`arr.join('')`|生成最终不可变 String|
|`length()` / `capacity()`|`str.length` / 内部 buffer 大小|capacity 可预分配减少扩容|

## StringJoiner

> **`StringJoiner` ≈ `Array.join()` + 自动处理空值/边界 + 可选的前后缀包装**

```java
StringJoiner sj = new StringJoiner(", ", "[", "]");

// 🔑 当没有添加任何元素时，输出默认空值而非 "[]"
sj.setEmptyValue("EMPTY");
System.out.println(sj); // EMPTY

// 添加了元素后，正常输出前后缀
sj.add("item");
System.out.println(sj); // [item]
```

