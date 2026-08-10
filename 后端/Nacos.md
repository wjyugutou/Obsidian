# Nacos：注册中心 + 配置中心

> Nacos（Dynamic Naming and Configuration Service）是阿里开源的服务发现、配置管理和服务管理平台。国内微服务领域首选，占 50%+ 市场份额。3.x 起定位升级为「面向 AI Agent 应用的注册中心」（MCP Registry / A2A 协议）。

## 核心概念

| 概念 | 说明 |
|---|---|
| **注册中心** | 服务提供者把地址（IP:Port）注册上去，消费者按服务名拉取实例列表，实现服务发现与负载均衡 |
| **配置中心** | 把配置从应用里抽离出来集中管理，修改配置**无需重启**，客户端实时生效（长轮询/gRPC 推送） |
| **namespace**（命名空间） | 最外层隔离单位，用于环境隔离（dev/test/prod）或租户隔离，默认 `public` |
| **group**（分组） | namespace 内的二级分组，默认 `DEFAULT_GROUP`，一般用于区分不同业务线 |
| **dataId** | 配置的唯一标识，命名规则：`${prefix}-${spring.profiles.active}.${file-extension}`（如 `application-prod.yaml`） |
| **服务实例** | 一个服务下的具体节点，带 IP、端口、权重、元数据 |
| **临时实例 / 持久实例** | 临时实例用**心跳**维持（客户端报心跳，超时即剔除，AP 模式）；持久实例靠**服务端主动探活**（CP 模式） |
| **AP / CP 模式** | 临时实例走 AP（可用性优先，Distro 协议）；持久实例走 CP（一致性优先，Raft 协议）。绝大多数场景用临时实例（AP） |
| **数据分级模型** | `namespace → group → service/dataId`，两级隔离先定清楚，避免后期重构 |

## 用途（解决什么问题）

1. **服务发现**：微服务多实例部署、动态扩缩容时，消费者无需硬编码地址，按服务名调用，Nacos 自动分发到健康实例
2. **健康检查**：实时剔除不健康实例，防止请求打到故障节点
3. **动态配置**：集中管理多环境配置；配置变更热更新，无需重新发布应用；支持版本回滚、灰度发布（金丝雀）
4. **服务治理基础**：权重路由、流量管理、服务元数据管理，配合 Sentinel/Dubbo 等实现限流、熔断
5. **AI 生态（3.x）**：作为 MCP Registry 管理 MCP Server、动态 Prompt，A2A 协议实现 Agent 之间互相发现与协作

## 使用场景

| 场景 | 说明 |
|---|---|
| Spring Cloud 微服务 | 服务注册/发现（`nacos-discovery`）+ 配置中心（`nacos-config`）双件套 |
| Dubbo / gRPC 服务 | 支持 RPC 类服务的注册发现，替代 Zookeeper |
| 多环境配置管理 | namespace 按环境隔离，一套代码多环境切换 |
| 配置热更新 | 开关、限流阈值、黑白名单等动态调整，无需发版 |
| 灰度发布 | 同 dataId 发布多版本配置（`Gray` 版本），按标签灰度 |
| 数据中心内网 DNS | 动态 DNS 服务，消除对厂商私有服务发现 API 的依赖 |
| AI 应用（3.x） | MCP Server 注册/管理/自动发现、Agent（A2A）注册与调用 |

## 常用方法/逻辑

### 1. 依赖引入

```xml
<!-- 服务注册发现 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<!-- 配置中心 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

### 2. 注册中心配置（application.yml）

```yaml
spring:
  application:
    name: order-service        # 应用名 = Nacos 服务名
  cloud:
    nacos:
      server-addr: 127.0.0.1:8848   # Nacos 地址
      username: nacos
      password: nacos
      discovery:
        namespace: dev               # 环境隔离，默认 public
        group: DEFAULT_GROUP
        ephemeral: true              # 临时实例（AP 模式）
```

- 启动即自动注册，无需 `@EnableDiscoveryClient`（Spring Cloud 2020+ 自动开启）
- 消费者调用：`@LoadBalanced RestTemplate` 或 OpenFeign 按服务名调用，底层自动选健康实例

### 3. 配置中心

```yaml
spring:
  config:
    import: nacos:application-dev.yaml?group=DEFAULT_GROUP   # 导入远程配置
  cloud:
    nacos:
      config:
        file-extension: yaml        # dataId 文件扩展名
        shared-configs:             # 共享配置（多服务共用）
          - data-id: common.yaml
            group: DEFAULT_GROUP
```

```java
@RefreshScope                              // 关键：配置变更后自动刷新 Bean
@Component
public class DynamicConfig {
    @Value("${order.timeout:5000}")         // 从 Nacos 配置读取，可热更新
    private int timeout;
}
```

- **dataId 自动匹配规则**：`${spring.application.name}-${profile}.${file-extension}`，如 `order-service-dev.yaml`
- 配置文件加载优先级：`shared-configs < extension-configs < 应用自身 dataId`

### 4. 原生 API（不依赖 Spring Cloud 封装）

```java
// NamingService：注册/发现/订阅
NamingService naming = NamingFactory.createNamingService("127.0.0.1:8848");
naming.registerInstance("order-service", "1.2.3.4", 8080);
List<Instance> instances = naming.getAllInstances("order-service");
Instance healthy = naming.selectOneHealthyInstance("order-service");  // 带负载均衡
naming.subscribe("order-service", event -> { /* 实例变化回调 */ });

// ConfigService：获取/发布/监听配置
ConfigService config = NamingFactory.createConfigService("127.0.0.1:8848");
String content = config.getConfig("dataId", "DEFAULT_GROUP", 5000);
config.publishConfig("dataId", "DEFAULT_GROUP", "order.timeout=5000");
config.addListener("dataId", "DEFAULT_GROUP", new AbstractListener() {
    @Override public void receiveConfigInfo(String configInfo) { /* 热更新 */ }
});
```

### 5. 端口与通信（2.x+）

| 端口 | 用途 |
|---|---|
| `8848` | HTTP API（OpenAPI、控制台） |
| `9848` | 客户端 gRPC 长连接（2.x 起性能比 1.x HTTP 提升 10 倍） |
| `9849` / `7848` | 服务端间通信 / Raft 选举 |

## 注意事项

- **临时实例选 AP、持久实例选 CP**：日常微服务用临时实例；需要严格数据一致的场景（如集群选主）才用持久实例
- **命名空间与 Group 提前规划**：两级隔离是数据边界，后期迁移成本高
- 配置变更走**灰度**再全量：利用 Nacos 灰度发布能力，先小流量验证
- 敏感配置（密码、密钥）使用 `cipher-` 前缀 + 配置加密插件，不要明文存储
- Nacos 是内部组件，生产环境不要暴露公网，开启鉴权（`nacos.core.auth.enabled=true`）
