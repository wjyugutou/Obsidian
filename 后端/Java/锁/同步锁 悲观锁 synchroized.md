
典型的悲观锁实现

代表类 `synchronized`、`ReentrantLock`

## 作用

数据安全

## 原理

MonitorObject 

owner 持有锁的线程


## 案例

### Spring Bean 类 中的成员变量 = 共享资源 = 必须考虑线程安全

```java
@RestController
public class CounterController {
	// 统计接口嗲用次数
	// springboot 中 是单例, 每个 count 请求访问的都是同一个CounterController实例中的 counter
    private int counter = 0;
    
    @GetMapping("/count")
    public int count() {
        counter++;  // ❌ 你以为是单线程，其实是 200 个线程并发
        return counter;
    }
}
```