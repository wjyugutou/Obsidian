## Servlet

**Servlet 是 Java Web 的“原子级”请求处理规范。**  
如果把 Tomcat/Jetty 比作 Node.js Runtime (或 Bun)，那么 Servlet 就是 Java 世界里最底层的 `http.createServer((req, res) => {})` 回调函数的**标准化接口**。

|概念|Node.js / NestJS 生态|Java Servlet 生态|备注|
|:--|:--|:--|:--|
|运行时容器|Node / Bun|Tomcat / Jetty / Undertow|Servlet 不能独立运行，必须跑在容器里|
|核心接口|`(req, res) => void`|`javax.servlet.Servlet`|定义了生命周期和请求处理方法|
|请求/响应对象|`Request` / `Response`|`HttpServletRequest` / `HttpServletResponse`|强类型，基于 Stream|
|中间件机制|Express Middleware / Nest Guard|Filter|⚠️ 注意：Servlet Filter 才是洋葱模型的实现|
|路由分发|Express Router / Nest Controller|`url-pattern` 映射 / Spring DispatcherServlet|原生 Servlet 路由很原始|
|框架封装|NestJS / Express|Spring MVC / Jakarta EE|现代开发几乎不直接写 Servlet|

### 2. 核心本质：它解决了什么问题？

在 Servlet 出现之前（90年代），Java 写 Web 用的是 CGI，每个请求都要启动一个 JVM 进程，性能极差。

Servlet 的核心价值是：**“一次加载，多次服务”**。

- 容器启动时加载 Servlet 类并实例化（单例）。
- 每个请求进来，容器分配一个线程调用该实例的 `service()` 方法。
- 这就像 NestJS 中 Controller 默认是单例 Scope 一样。

### 3. 和你熟知的“洋葱模型”的关系

这是转 Java 最容易混淆的点：

- **Servlet 本身不是洋葱模型**。它只是一个 `init() -> service() -> destroy()` 的生命周期接口。
- **Servlet Filter 才是洋葱模型**。
    - Filter 链（FilterChain）的执行顺序完全等同于 Express/Koa 的中间件栈。
    - `chain.doFilter(req, res)` 就等于 NestJS/Express 里的 `next()`。
    - Spring Security 的认证授权、Spring MVC 的字符编码过滤，全都是基于 Servlet Filter 实现的。

```java
// 这就是 Java 版的 next()
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
    // 前置逻辑 (Before next())
    System.out.println("Request In");
    
    chain.doFilter(req, res); // ✅ 等价于 await next()
    
    // 后置逻辑 (After next())
    System.out.println("Response Out");
}
```

## Tomcat

**Tomcat 是 Servlet 规范的“官方参考实现” + 一个轻量级 Web 服务器。**  
如果用你的技术栈类比：**Tomcat ≈ Bun Runtime + 内置的 HTTP Server**。它负责把网络字节流变成 `HttpServletRequest`，再把你写的 Servlet 跑起来。