思想 面向切面: 把日志、事务、权限等"横切关注点"从业务代码中剥离, 通过代理统一织入

## 代理机制

Spring AOP 基于**动态代理**, 运行期生成代理对象替代原 Bean:

| 方式 | 条件 | 原理 |
|---|---|---|
| JDK 动态代理 | Bean 实现了接口 | 生成接口的代理类 (Proxy.newProxyInstance) |
| CGLIB 代理 | 没实现接口 (或 SpringBoot 默认) | 生成目标类的子类, 覆写方法 |

SpringBoot 2.x 起默认 CGLIB (proxyTargetClass=true), 无需配置。
因为代理是子类/接口实现, 所以:
- 切面方法不能是 `private` / `final`
- 同类内部 `this.xxx()` 调用不走代理, 事务/切面会失效 (要注入自身或用 AopContext.currentProxy())

## 通知类型 (Advice)

| 通知 | 时机 | 类比 |
|---|---|---|
| `@Before` | 方法执行前 | 前置 |
| `@AfterReturning` | 正常返回后 | 后置 |
| `@AfterThrowing` | 抛异常后 | 异常 |
| `@After` | 无论结果 (finally) | 收尾 |
| `@Around` | 全包围, 可控制是否执行/改参数/改返回值 | 最强, 一般用它一个就够 |

## 最小示例

```java
@Aspect
@Component
public class LogAspect {
    // 切点: 哪个包下所有方法
    @Pointcut("execution(* com.hmdp.service.*.*(..))")
    private void pt() {}

    @Around("pt()")
    public Object log(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        System.out.println(pjp.getSignature().getName() + " 耗时 " +
                (System.currentTimeMillis() - start) + "ms");
        return result;
    }
}
```

## 与 NestJS 对照

NestJS 没有 AOP 概念, 但用"装饰器 + 管道/守卫/拦截器/过滤器"四件套实现了同样的横切关注点:

| Spring AOP | NestJS | 时机 |
|---|---|---|
| `@Around` (最接近) | `Interceptor` (`@UseInterceptors`) | 包住整个执行, 可改响应 |
| `@Before` | `Guard` (如 AuthGuard) | 请求前鉴权 |
| `@Before` (参数处理) | `Pipe` (ValidationPipe) | 请求前转换/校验参数 |
| `@AfterThrowing` | `ExceptionFilter` | 异常统一处理 |
| `@After` | Middleware | 收尾/通用逻辑 |

本质差异: Spring 靠**运行期代理**无侵入织入; NestJS 靠**装饰器显式声明**, 看得见但侵入性更强。业务代码若大量出现 @UseGuards/@UseInterceptors, 就是在手工做 AOP。

参考: [[Spring常用注解]], [[SpringMVC]]
