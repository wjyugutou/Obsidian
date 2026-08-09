Session 就像接口

Memory/Redis 就是 实现

┌─────────────────────────────────────┐
│         Session API (抽象层)          │  ← HttpSession / express-session / Django session
│   getAttribute() / setAttribute()    │
├─────────────────────────────────────┤
│       Session Store (接口层)          │  ← Spring Session / connect-redis / django-redis
├──────────┬──────────┬───────────────┤
│  Redis   │  JDBC    │  内存(Map)     │  ← 具体存储后端
│ (推荐)   │ (MySQL)  │ (仅开发用)     │
└──────────┴──────────┴───────────────┘  

### 什么情况下 Session ID 不同？

| 场景                       | Session ID | 原因                    |
| :----------------------- | :--------- | :-------------------- |
| 同一浏览器，同域，不同标签页           | 相同         | Cookie 按域共享           |
| 同一浏览器，不同域名               | 不同         | Cookie 隔离             |
| 无痕/隐私模式窗口                | 不同         | 独立 Cookie 存储          |
| 不同浏览器（Chrome vs Firefox） | 不同         | 完全隔离                  |
| 移动端 App WebView          | 通常不同       | 独立 CookieManager      |
| 手动清除 Cookie 后刷新          | 新 ID       | 旧 Session 丢失，服务端创建新会话 |

| 问题                        | 答案                                     |
| :------------------------ | :------------------------------------- |
| 同浏览器不同标签页 Session ID 一样吗？ | 一样（同域前提下）                              |
| 那它们共享同一个 Session 对象吗？     | 逻辑上共享数据，物理上是独立副本                       |
| 会带来什么问题？                  | 并发写覆盖、状态不一致                            |
| 如何应对？                     | Session 尽量只读；可变状态用 Redis 原子操作；前端跨标签页同步 |
