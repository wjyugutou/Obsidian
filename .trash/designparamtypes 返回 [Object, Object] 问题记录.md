现象
// app.controller.ts
constructor(
  private loggerService: LoggerService,
  @Inject('StringToken') private useValueService: UseValueService,
) {}

// 用 @Inject 的可以，没用 @Inject 的不行
Reflect.getMetadata(DESIGN_PARAMTYPES, AppController)
// 实际：[Function: Object, Function: Object]
// 期望：[class LoggerService, Function: Object]
根因
emitDecoratorMetadata 在编译时 emit design:paramtypes，需要运行时构造函数引用。如果类型是通过 import type 导入的，编译后该 import 被完全擦除，TypeScript 无法拿到运行时值，只能 fallback 到 Object。
// ❌ 运行时擦除，design:paramtypes 拿不到 LoggerService
import type { LoggerService } from './logger.service'

// ✅ 保留运行时引用，design:paramtypes 正确
import { LoggerService } from './logger.service'
为什么 @Inject 不受影响
@Inject('StringToken') 走的是自主存的 INJECTED_TOKENS 元数据（@Inject 装饰器手动 Reflect.defineMetadata），不依赖编译器的 design:paramtypes。所以无论是否 import type，@Inject 的 token 都能正确工作。
对应 resolveDependencies 逻辑：
const injectedTokens = Reflect.getMetadata(INJECTED_TOKENS, Clazz) || []  // 有 @Inject 则正确
const constructorParams = Reflect.getMetadata(DESIGN_PARAMTYPES, Clazz) as any[] || []  // 依赖运行时类型

return constructorParams.map((param, index) => {
  return this.providersMap.get(injectedTokens[index] ?? param)  // @Inject 优先，否则 fallback 到 design:paramtypes
})
解决方案
方案 A（推荐）：给所有参数加 @Inject，不再依赖 design:paramtypes fallback
import type { LoggerService } from './logger.service'

export class AppController {
  constructor(
    @Inject(LoggerService) private loggerService: LoggerService,
    @Inject('StringToken') private useValueService: UseValueService,
  ) {}
ESLint consistent-type-imports 规则保持开启，不需要特殊处理。
方案 B：改用普通 import + ESLint disable
// eslint-disable-next-line @typescript-eslint/consistent-type-imports
import { LoggerService } from './logger.service'
本质
design:paramtypes 是 TypeScript 编译器在编译时根据可见的运行时引用 emit 的。当某种类型只在类型空间存在（import type、接口、type alias），编译器在 emit design:paramtypes 时无法产生该类型的运行时引用，只能用 Object 做 fallback。这是 emitDecoratorMetadata 的设计限制，不是 bug。