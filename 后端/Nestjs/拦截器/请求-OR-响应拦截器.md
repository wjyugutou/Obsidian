---
created: 2026-07-30
updated: 2026-07-30
tags:
  - nestjs
  - interceptor
---
NestJS **没有**"请求拦截器 / 响应拦截器"两种类。同一个 `intercept()` 方法同时覆盖两个阶段，区分只看代码写在 `next.handle()` 的哪一侧：

```ts
intercept(context, next) {
  // 【请求阶段】next.handle() 之前的代码 —— controller 执行前跑
  //   例：记请求时间、读 token、查缓存命中则 of(cached) 直接返回不走 controller

  return next.handle().pipe(
    // 【响应阶段】pipe 里的操作 —— controller 返回后跑
    map((data) => ...),       // 转换响应
    tap(() => ...),           // 副作用
    catchError((err) => ...), // 异常映射
  );
}
```

**根因**：`next.handle()` 返回的是冷 Observable，只有订阅时才触发 controller。所以 `next.handle()` 之前的代码先跑（请求阶段），`.pipe()` 里的操作符在 controller 产生值后跑（响应阶段）。

本项目 4 个拦截器全部只在响应阶段做事（`next.handle()` 之前无代码），因为它们职责都是响应转换：

| 拦截器 | 请求阶段代码 | 响应阶段代码 |
| --- | --- | --- |
| ResTransform | 无 | `map(data => {code,data,msg})` |
| Dict | 无 | `map(data => 翻译)` |
| DateTransform | 无 | `map(data => 格式化 Date)` |
| ZodSerializer | 无（继承 nestjs-zod） | 响应 schema 校验/序列化 |

**请求阶段典型职责**：请求日志、缓存命中短路（`of(cached)` 不调 `next.handle()`）、限流、注入 traceId。

**关联点**：做"请求→响应"关联（如耗时统计）必须在请求阶段用闭包变量存住（如 `const now = Date.now()`），响应阶段在 `pipe` 里引用——响应阶段拿不到"执行前"快照。
