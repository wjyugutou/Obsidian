# Tomcat：Servlet 容器 / Web 服务器

> Apache Tomcat 是 Java 生态最主流的 **Servlet 容器**：负责把 HTTP 请求交给 Servlet 处理、管理 Servlet/JSP 生命周期，也是 Spring Boot 默认内嵌的 Web 服务器。最新版本：11.0.x（Servlet 6.1，需 Java 17+）、10.1.x（Java 11+）、9.0.x（Java 8+）。

## 核心概念

| 概念 | 说明 |
|---|---|
| **Servlet 容器** | 管理 Servlet 的创建、初始化、调用、销毁（生命周期），把 HTTP 请求解析后路由给对应 Servlet，再把结果写回响应 |
| **Connector（连接器，Coyote）** | 负责接收/解析 HTTP 请求，把请求交给 Container；对应 `server.xml` 里的 `<Connector>`（端口即在此配置） |
| **Container（容器，Catalina）** | 负责处理请求，核心引擎；对应 `<Engine>`、`<Host>`、`<Context>`、`<Wrapper>` |
| **容器层级** | `Engine`（整个引擎）→ `Host`（虚拟主机）→ `Context`（一个 Web 应用）→ `Wrapper`（一个 Servlet），请求按层级逐级分发 |
| **请求处理流程** | `Connector 接收请求 → Engine → Host（按域名选虚拟主机）→ Context（按路径前缀选应用）→ Wrapper（按 URL 映射选 Servlet）→ Servlet 执行业务 → 响应原路返回` |
| **I/O 模型** | 默认 **NIO**；还有 NIO2、APR（配合 Tomcat Native，性能最好）。BIO 已废弃。HTTP/1.1、HTTP/2 均支持 |
| **类加载器隔离** | 每个 Web 应用（Context）有独立类加载器，应用之间类不互通；`WEB-INF/lib` 与容器 `lib/` 目录分离，遵循"子优先"加载 |
| **Session 管理** | 默认内存 Session（重启丢失）；集群场景可配 Session 持久化或 Redis 共享 |

### 版本与命名空间（易踩坑）

| Tomcat | Servlet 规范 | Java 包名 | Java 要求 |
|---|---|---|---|
| **9.0.x** | Servlet 4.0 | `javax.servlet.*` | Java 8+ |
| **10.1.x** | Servlet 6.0/6.1 | `jakarta.servlet.*` | Java 11+ |
| **11.0.x** | Servlet 6.1 | `jakarta.servlet.*` | Java 17+ |

> 10.x 起包名从 `javax` 换成 `jakarta`，老代码（`import javax.servlet.*`）直接迁移会编译失败。Spring Boot 3.x 内嵌的是 Tomcat 10.x，Spring Boot 2.x 内嵌 9.x。

### 目录结构

| 目录 | 用途 |
|---|---|
| `bin/` | 启动/停止脚本（`startup.sh`、`shutdown.sh`） |
| `conf/` | 核心配置：`server.xml`（端口/连接器）、`web.xml`（全局 Servlet 配置）、`context.xml` |
| `webapps/` | Web 应用部署目录，丢 war 包即自动部署 |
| `logs/` | 运行日志（`catalina.out`、`localhost.log`） |
| `lib/` | 容器级公共 jar，所有应用共享 |

## 用途（解决什么问题）

1. **运行 Web 应用**：Servlet/JSP/Filter/Listener 的运行时环境，把 Java 代码变成可被 HTTP 访问的服务
2. **协议与业务解耦**：HTTP 解析（Connector）与业务处理（Container）分离，开发者只写 Servlet，不用关心网络细节
3. **静态资源服务**：内置 DefaultServlet，可托管 HTML/CSS/JS/图片
4. **Spring Boot 内嵌运行时**：现代微服务默认以可执行 jar 内嵌 Tomcat 运行，无需独立安装

## 使用场景

| 场景 | 说明 |
|---|---|
| 传统企业应用（war 部署） | 老项目/政府金融系统，打包成 `xxx.war` 放进 `webapps/` 或通过 manager 上传部署 |
| Spring Boot 内嵌 | 现代微服务默认内嵌 Tomcat，独立部署/容器化（Docker）都无需外部 Tomcat |
| Nginx + Tomcat 组合 | Nginx 扛静态资源、HTTPS、负载均衡，Tomcat 只处理动态请求（前后端分离标配） |
| 多应用共享实例 | 一个 Tomcat 部署多个应用：虚拟主机（Host）隔离域名、Context 隔离应用路径 |
| JSP 技术栈项目 | JSP 页面编译（Jasper 引擎）需要 Tomcat，国内老系统仍有存量 |
| 集群部署 | 多实例 + Session 共享（Redis/持久化），配合 Nginx 负载均衡 |

## 常用方法/逻辑

### 1. 独立部署（server.xml 关键配置）

```xml
<!-- conf/server.xml -->
<!-- 连接器：端口 + 协议 + 参数 -->
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"          <!-- 连接超时 ms -->
           redirectPort="8443" />

<!-- 线程池：共享给多个连接器 -->
<Executor name="tomcatThreadPool"
          namePrefix="catalina-exec-"
          maxThreads="400"      <!-- 最大工作线程 -->
          minSpareThreads="20" /> <!-- 最小空闲线程 -->
```

- 常用参数：`maxThreads`（默认 200）、`acceptCount`（队列长度，默认 100）、`maxConnections`（最大连接数）
- 连接器指定线程池：`<Connector executor="tomcatThreadPool" .../>`
- 虚拟主机：`<Host name="www.example.com" appBase="webapps"/>`，一个引擎下可配多个 Host

### 2. 应用部署方式

| 方式 | 操作 | 说明 |
|---|---|---|
| 目录部署 | 解压 war 到 `webapps/`，目录名 = 访问路径 | 开发调试方便 |
| war 热部署 | 直接复制 `xxx.war` 到 `webapps/` | Tomcat 自动解压并部署，更新时**重命名旧文件或删除 war** 才会重新加载 |
| manager 管理台 | 访问 `http://localhost:8080/manager` | 需先在 `conf/tomcat-users.xml` 配置 `manager-gui` 角色账号 |
| 编程式部署 | `/manager/text/deploy?path=/app&war=file:...` | 脚本化/自动化部署用 |

### 3. Spring Boot 内嵌 Tomcat 配置

```yaml
server:
  port: 8080
  tomcat:
    threads:
      max: 200            # 最大工作线程
      min-spare: 10       # 最小空闲线程
    accept-count: 100     # 等待队列长度（超出后拒绝）
    max-connections: 8192 # 最大连接数
    connection-timeout: 20s
    max-http-form-post-size: 2MB
    compression:          # 响应压缩
      enabled: true
      min-response-size: 1024
  servlet:
    context-path: /api    # 全局路径前缀
```

```java
// 编程式调整（BeanPostProcessor / 自定义实现）
@Bean
public WebServerFactoryCustomizer<TomcatServletWebServerFactory> tomcatCustomizer() {
    return factory -> factory.addConnectorCustomizers(
        connector -> connector.setProperty("maxKeepAliveRequests", "100"));
}
```

### 4. 性能调优要点

1. **线程池**：`maxThreads` 不是越大越好，与机器核数、业务耗时匹配（经验值：CPU 密集 ≈ 核数 × 2）
2. **超时与队列**：`connectionTimeout` 过长会耗尽连接；`acceptCount` 太小高峰直接拒绝
3. **日志**：`AccessLogValve` 开启访问日志，排障必用
4. **静态资源**：交给 Nginx，或开启压缩 + 缓存头，别让 Tomcat 扛静态
5. **APR/Native**：高并发场景装 Tomcat Native 提升性能（OpenSSL 支持）

### 5. 常用运维命令

```bash
bin/startup.sh        # 启动
bin/shutdown.sh       # 停止
# 日志：tail -f logs/catalina.out
```

## 注意事项

- **版本选择**：新项目用 Spring Boot 内嵌 Tomcat 即可；独立部署选 10.1.x（Servlet 6.0，Java 11+）起步，老项目保持 9.0.x（javax 包名）
- **安全**：生产不要用默认端口直接暴露公网，前端必须有 Nginx/网关；`manager` 控制台默认无密码保护，需配置 `tomcat-users.xml` 且勿用弱口令
- **及时升级**：Tomcat 是公网攻击目标，关注安全公告（如 CVE-2026-55956，修复版本 9.0.119 / 10.1.56 / 11.0.23+）
- **Session 丢失**：默认内存 Session 重启即丢，生产集群必须接 Redis/外部存储
- **不要部署在容器里再套容器**：Docker 部署时应用已内嵌 Tomcat（Spring Boot jar），无需再装独立 Tomcat
