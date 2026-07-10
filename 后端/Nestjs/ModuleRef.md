### 1. 在非类上下文中获取依赖（最常见）

当你处于**无法使用构造函数注入**的环境中时：
- **工具函数 / 静态方法**：普通函数不是由 NestJS 管理的，无法使用 `@Inject()`
- **事件监听器回调**：如 EventEmitter2 的异步回调、消息队列消费者中的闭包
- **定时任务 (Cron)**：某些调度库的回调不在 DI 容器内
- **自定义 Provider 工厂函数**：`useFactory` 中需要条件性获取其他服务

### 2. 动态/条件性解析依赖

当你要注入的服务**在编译期无法确定**，需要根据运行时条件决定：

### 3. 解决循环依赖

当两个服务互相依赖，且 `forwardRef()` 无法满足需求时（例如在 `onModuleInit` 之外需要延迟解析）：

### 4. 创建独立的请求作用域实例

默认 `moduleRef.get()` 返回的是**单例**。如果你需要获取一个**新的 REQUEST 作用域**实例（而非共享单例），必须传入 `{ strict: false }` 并配合上下文：

```ts
// 获取一个绑定到当前请求上下文的瞬态/请求作用域服务
const requestScopedService = await this.moduleRef.get(
  RequestScopedService,
  contextId, // 通过 ContextIdFactory.getByRequest(request) 获取
  { strict: false }
);
```