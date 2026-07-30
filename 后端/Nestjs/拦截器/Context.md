---
created: 2026-07-26
updated: 2026-07-30
tags:
  - nestjs
  - interceptor
  - context
---
## NestInterceptor.intercept.context

| 方法                         | 返回内容               | 对应装饰器位置                     | 代码示例                  |
| -------------------------- | ------------------ | --------------------------- | --------------------- |
| **`context.getHandler()`** | **方法（Method）** 的引用 | 贴在**方法**上的 `@DictTranslate` | `@Get()` 下面的那个函数      |
| **`context.getClass()`**   | **类（Class）** 的构造函数 | 贴在**类**上的 `@DictTranslate`  | `OrderController` 这个类 |
