## `design:paramtypes` 返回 `[Function: Object, Function: Object]` 问题

### 现象

```typescript
constructor(
  private loggerService: LoggerService,
  @Inject('StringToken') private useValueService: UseValueService,
) {}

Reflect.getMetadata(DESIGN_PARAMTYPES, AppController)
// 实际：[Function: Object, Function: Object]
// 期望：[class LoggerService, Function: Object]
```

- 有 `@Inject` 的参数正常
- 依赖 `design:paramtypes` fallback 的参数拿到的是 `Object`

### 根因

`emitDecoratorMetadata` 在编译时 emit `design:paramtypes`，需要**运行时构造函数引用**。`import type` 在编译后完全擦除，TypeScript 无法在该位置拿到 `LoggerService` 的运行时值，fallback 到 `Object`。

```typescript
import type { LoggerService } from './logger.service'  // ❌ 运行时擦除
import { LoggerService } from './logger.service'       // ✅ 保留运行时引用
```

### 为什么 `@Inject` 不受影响

`@Inject` 走的是自定义 `INJECTED_TOKENS` 元数据（`Reflect.defineMetadata` 手动存储），不依赖编译器 emit 的 `design:paramtypes`。无论是否 `import type` 都正确。

对应 `resolveDependencies` 逻辑：

```typescript
const injectedTokens = Reflect.getMetadata(INJECTED_TOKENS, Clazz) || []
const constructorParams = Reflect.getMetadata(DESIGN_PARAMTYPES, Clazz) || []

return constructorParams.map((param, index) => {
  return this.providersMap.get(injectedTokens[index] ?? param)
  //                          ↑ @Inject 优先    ↑ design:paramtypes fallback
})
```

### 解决方案

#### 方案 A（推荐）：全局用 `@Inject`，彻底消除对 `design:paramtypes` 的依赖

```typescript
import type { LoggerService } from './logger.service'

export class AppController {
  constructor(
    @Inject(LoggerService) private loggerService: LoggerService,
    @Inject('StringToken') private useValueService: UseValueService,
  ) {}
}
```

- ESLint 规则兼容，无需特殊处理
- DI 完全由显式 token 驱动

#### 方案 B：普通 `import` + ESLint disable

```typescript
// eslint-disable-next-line @typescript-eslint/consistent-type-imports
import { LoggerService } from './logger.service'
```

### 本质

`design:paramtypes` 是 TypeScript 编译期的产物，只对编译器在编译时能解析到的**运行时值**有效。`import type`、接口（`interface`）、类型别名（`type`）等纯类型结构在运行时不存在，编译器在 emit `design:paramtypes` 时无法产生对应引用，只能用 `Object` 做 fallback。这是 `emitDecoratorMetadata` 的设计限制，不是 bug。
