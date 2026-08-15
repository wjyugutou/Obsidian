
如
```yaml
app:
  executors:
    cache-rebuild:
      core-size: 2
      max-size: 8
      queue-capacity: 200
      prefix: cache-rebuild-
    email-sender:
      core-size: 1
      max-size: 2
      queue-capacity: 50
      prefix: email-
```

executors springboot 不认识

```java
@Data
@ConfigurationProperties(prefix = "app.executors")
public class ExecutorProps {
    private Map<String, PoolProps> executors = new HashMap<>();  // 每个 key 一个线程池
}

@Bean
public Map<String, ThreadPoolTaskExecutor> taskExecutors(ExecutorProps props) {
    // 循环 props.executors 构建多个，key 即 bean 名
}
```

职责链条是：
yml (app.executors.*)
  → ① ExecutorProps 绑定解析成 Map   (纯数据转换)
  → ② taskExecutors 遍历 Map 创建多个 ThreadPoolTaskExecutor 并注册为 bean  (真正实例化)
  → ③ 业务代码 @Resource(name="cacheRebuildExecutor") 注入使用 (首次提交任务才真正起线程)
一句话：ExecutorProps 是"翻译官"（yml → 数据），taskExecutors 是"工厂"（数据 → 线程池对象），线程本身要等业务真正提交任务才懒启动。

