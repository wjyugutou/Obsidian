思想 SpringBoot 大部分功能 = 注解开关 + 约定配置, 注解是它的"装饰器"

## Bean 注册

| 注解 | 说明 |
|---|---|
| `@Component` | 通用组件, 注册为 Spring Bean |
| `@Service` | 业务层, 语义化的 @Component |
| `@Repository` | 持久层, 语义化的 @Component |
| `@Controller` / `@RestController` | 控制层, 语义化的 @Component |
| `@Configuration` | 配置类, 内部 @Bean 方法产出的对象也会注册为 Bean |
| `@Bean` | 方法级, 声明一个 Bean 交给容器管理(常用于第三方类) |

## 依赖注入

| 注解                                      | 说明                                  |
| --------------------------------------- | ----------------------------------- |
| `@Autowired`                            | 按类型注入, 类比 NestJS 构造函数注入             |
| `@Qualifier("名称")`                      | 同类型多 Bean 时指定名称, 配合 @Autowired      |
| `@Resource`                             | 按名称注入 (javax/jakarta 标准)            |
| `@Value("${xx.yy}")`                    | 注入配置文件属性, 类比 NestJS 的 ConfigService |
| `@ConfigurationProperties(prefix="xx")` | 批量绑定配置到 POJO                        |

## 配置与条件

| 注解 | 说明 |
|---|---|
| `@SpringBootApplication` | 启动器 = @Configuration + @EnableAutoConfiguration + @ComponentScan |
| `@Profile("dev")` | 环境切换, 类比 NestJS 的 process.env.NODE_ENV |
| `@ConditionalOnClass` / `@ConditionalOnMissingBean` | 条件装配, 自动配置的核心机制 |

## Web 层

见 [[SpringMVC]] 对照表, 补充:

| 注解 | 说明 |
|---|---|
| `@CookieValue` | 取 Cookie |
| `@CrossOrigin` | 跨域允许 |
| `@Validated` / `@Valid` | 触发参数校验 (配合 jakarta.validation 注解: @NotNull @NotBlank @Min 等) |
| `@SessionAttribute` | 取 Session |

## 事务 / 异步 / 定时

| 注解                                 | 说明                                                   |
| ---------------------------------- | ---------------------------------------------------- |
| `@Transactional`                   | 方法/类级事务, 默认仅对 RuntimeException 回滚                    |
| `@Async` + `@EnableAsync`          | 异步执行, 类比 NestJS @nestjs/bull 或独立异步任务                 |
| `@Scheduled` + `@EnableScheduling` | 定时任务, 类比 NestJS @Cron() (需要 @nestjs/schedule)        |
| `@PostConstruct` / `@PreDestroy`   | 初始化 / 销毁回调, 类比 NestJS OnModuleInit / OnModuleDestroy |

## 缓存

| 注解 | 说明 |
|---|---|
| `@EnableCaching` | 开关 |
| `@Cacheable` | 先查缓存, 未命中则执行方法并缓存 |
| `@CacheEvict` | 清缓存 |
| `@CachePut` | 更新缓存 |

## 切面

`@Aspect`、`@Pointcut`、`@Before`、`@After`、`@AfterReturning`、`@AfterThrowing`、`@Around` —— 见 [[AOP]]

## Spring 注解 ↔ NestJS 装饰器对照表

| Spring | NestJS | 说明 |
|---|---|---|
| `@Component` / `@Service` / `@Repository` | `@Injectable()` | 注册可注入依赖 |
| `@Controller` / `@RestController` | `@Controller()` | 控制器 |
| `@GetMapping` 等 | `@Get()` 等 | 路由方法 |
| `@PathVariable` | `@Param('id')` | 路径参数 |
| `@RequestParam` | `@Query('key')` | 查询参数 |
| `@RequestBody` | `@Body()` | 请求体 |
| `@RequestHeader` | `@Headers('key')` | 请求头 |
| `@Autowired` / 构造器注入 | 构造器注入 | DI |
| `@Transactional` | TypeORM 的 `@Transaction()` 或 DataSource.transaction() | 事务 |
| `@Valid` + 校验注解 | DTO + class-validator | 参数校验 |
| `@Scheduled` | `@Cron()` | 定时任务 |
| `@Aspect` + `@Around` | `@UseInterceptors()` + `Interceptor` | 横切逻辑 |
| `@RestControllerAdvice` | `@Catch()` + ExceptionFilter | 全局异常 |
| `@ConfigurationProperties` | ConfigModule 的 validate/load | 配置绑定 |
