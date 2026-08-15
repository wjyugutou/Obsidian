思想 分层

## Entity

mysql有一张表 这里就有一个类,有两个表就两个类

## Dao

写SQL,操作数据库

## Service

业务代码, 对应 nestjs的Service

## Controller

1. 给前端调的,对应 nestjs的Controller
2. 调用service的代码, 通过 @Autowired 注入, 类比 nestjs 构造函数注入（constructor(private svc: XxxService)）

## Spring MVC ↔ NestJS 对照表

| Spring MVC | NestJS | 说明 |
|---|---|---|
| `@RestController` / `@Controller` | `@Controller()` | 类装饰器 |
| `@GetMapping` `@PostMapping` `@PutMapping` `@DeleteMapping` | `@Get()` `@Post()` `@Put()` `@Delete()` | 方法装饰器 |
| `@RequestMapping("/path")` | `@Controller('path')` 或方法级 `@Get('path')` | 路径前缀 |
| `@PathVariable` | `@Param('id')` | URL 路径参数 |
| `@RequestParam` | `@Query('key')` | 查询参数 |
| `@RequestBody` | `@Body()` | 请求体 |
| `@RequestHeader` | `@Headers('token')` | 请求头 |
| `HttpServletRequest` / `HttpServletResponse` | `@Req()` / `@Res()` | 原生请求/响应对象 |
| `@Autowired` 依赖注入 | 构造函数注入（`constructor(private svc: XxxService)`） | DI 机制 |
| `HandlerInterceptor`（登录拦截） | `Guard`（如 `AuthGuard`） | 请求前置拦截，黑马点评的登录拦截就是 Guard |
| `@Valid` + 参数校验 | `ValidationPipe`（配合 class-validator DTO） | 请求校验 |
| `@ExceptionHandler` / `@RestControllerAdvice`（全局异常） | `ExceptionFilter`（`@Catch()` + `@UseFilters`） | 全局异常处理 |
| `Filter`（Servlet 过滤器） | `Middleware`（`@Middleware()`） | 通用前置逻辑 |
