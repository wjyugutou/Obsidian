# RuoYi-Vue 从零实现计划

> 目标：从零实现 RuoYi-Vue 后端全部功能（JDK 17 + Spring Boot 4.0.6 + MyBatis + Redis + MySQL），用官方 ruoyi-ui 前端验证。
> 前提：会 Spring Boot 基础（能写简单 REST + MyBatis），未做过完整项目。
> 参考资料：本地仓库 /home/wwwj/workspace/wwwj/RuoYi-Vue（官方 3.9.2）

## 总体策略

- **依赖顺序先行**：`common → system → framework → quartz/generator → admin`，下层模块是上层的地基
- **接口契约对齐**：官方前端只认 `{code, msg, data}` 返回格式和固定字段名，每个模块做完必须用对应前端页面点一遍验证
- **先跑通再完善**：早期关掉验证码（`sys.account.captchaEnabled=false`）降低联调门槛，功能通了再开
- **别照抄**：每个模块先自己写，写不动再看官方源码对应文件，理解后合上重写

## 工程结构

```
RuoYi-Vue
├── ruoyi-admin      # 启动入口 + WebController（依赖所有模块）
├── ruoyi-framework  # Spring Security、AOP、拦截器、配置（依赖 system+common）
├── ruoyi-system     # 用户/角色/菜单/部门/字典/日志等业务（依赖 common）
├── ruoyi-common     # 工具类、常量、统一返回、异常、分页（无内部依赖）
├── ruoyi-quartz     # 定时任务（依赖 common）
├── ruoyi-generator  # 代码生成器（依赖 common+system）
└── ruoyi-ui         # 官方 Vue2 前端（只作验证工具）
```

## 阶段 0：环境与工程搭建（半天）

| 事项 | 学习点 |
|---|---|
| 装齐 JDK 17 / Maven / MySQL 8 / Redis | 无 |
| 执行 `sql/ry_20260417.sql` 建库建表 | 先通读表结构，理解 24 张表的职责 |
| pnpm 启动官方 `ruoyi-ui`（登录页能出来即可） | 前端只作为验证工具 |
| 手写 parent `pom.xml`：模块声明 + 统一版本管理 | Maven 多模块、dependencyManagement 与 dependencies 区别 |

**验证**：`mvn clean package` 通过；五个子模块依赖图清晰。

## 阶段 1：通用层 common（2~3 天）

| 事项 | 学习点 |
|---|---|
| `AjaxResult` + 错误码枚举：统一返回 `{code,msg,data}` | 前后端契约 |
| `GlobalExceptionHandler`：全局异常 → 统一 JSON | @RestControllerAdvice |
| 基础工具类（`StringUtils`、`DateUtils`、`SecurityUtils` 占位、`uuid`） | 只写用到的，别照抄全量 |
| `TableDataInfo` + 分页基类 `BaseEntity` | 分页契约 `total/rows` |

**验证**：写个临时 controller 手动造异常，看返回结构。

## 阶段 2：认证 + RBAC 权限（核心，1~2 周）

按依赖顺序拆成 7 步，**每一步都可在登录后由前端页面验证**：

1. **登录流程**：`SysLoginService` — 验证码校验（Redis 存取）→ 用户状态检查 → 密码 BCrypt 校验 → 生成 JWT
2. **Token 服务**：`TokenService` — JWT 生成/解析/刷新 + token 存 Redis（`login_tokens:`），学 JWT 结构（header.payload.signature）与 Redis 存登录态的取舍
3. **Security 配置**：`SecurityConfig` — 无状态会话、放行白名单（/login、/captchaImage、静态资源）、认证失败/未授权处理
4. **认证过滤器**：`JwtAuthenticationTokenFilter` 加入过滤器链，每次请求解析 token 并注入 `LoginUser` 到 SecurityContext
5. **权限模型落地**（学 RBAC 精髓，前端菜单/权限能否点亮全靠这步）：
   - 用户 ↔ 角色 ↔ 菜单 三表 + 关联表，部门/岗位树形结构（`treeSelect`、递归查询）
   - `@PreAuthorize` 注解 + 自定义权限校验（`ss.hasPermi('system:user:list')`）
   - `SecurityUtils.getLoginUser()` 获取当前用户
6. **动态路由菜单**：`getRouters` 按用户权限返回菜单树 → 前端侧边栏渲染
7. **登出**：删除 Redis token + 记录登录日志

**验证**：官方前端登录成功 → 用不同角色账号登录看菜单权限差异 → 拦截器里打断点理解请求全链路。

## 阶段 3：通用业务模块（1 周，每个模块半天~1 天）

按「学习价值从高到低」排序：

| 顺序 | 模块 | 学习点 |
|---|---|---|
| 1 | 操作日志 | AOP 切面 + 自定义注解 `@Log` + 异步保存（先学 AOP，后续全用得上） |
| 2 | 登录日志 | 登录流程里埋点（复用阶段 2 的登录服务） |
| 3 | 字典管理 | 类型/数据两张表 + `@Dict` 注解数据翻译（前端下拉全依赖它） |
| 4 | 参数配置 | 配置的增删改查 + 缓存（`sys_config:`） |
| 5 | 岗位管理 | 与用户关联（用户↔岗位 关联表） |
| 6 | 通知公告 | 简单 CRUD，练手 |
| 7 | 文件上传 | 本地存储 + 配置路径 + 访问映射 |
| 8 | Excel 导入导出 | POI + 自定义注解反射（`@Excel`） |

**验证**：每个模块做完，用前端「系统管理」对应页面走一遍增删改查 + 导出。

## 阶段 4：外围扩展（1 周）

| 顺序 | 模块 | 学习点 |
|---|---|---|
| 1 | 在线用户监控 | 遍历 Redis 中 `login_tokens:*` 的 key（理解 token 集中管理的价值） |
| 2 | 定时任务 Quartz | 任务表 + 动态增删改/暂停恢复 + 调 bean 方法（反射） |
| 3 | 服务监控 | Druid 监控页（DB）、Redis 监控、服务信息（`oshi`） |
| 4 | 缓存监控 | Redis 内存/键数统计 |
| 5 | WebSocket | 在线用户强退通知（后端推消息） |
| 6 | 代码生成器 | 读表结构 → Velocity 模板生成代码（最复杂，可选做简化版） |

## 阶段 5：收尾（1 天）

- 限流/防重复提交（AOP + Redis，可选）
- 打包部署：后端 jar + 前端 build 后 nginx 反代，`bin/` 脚本
- 对照官方源码做差异 review：**所有模块完成后，逐文件 diff 官方实现**，找出你没想到的设计点（这是最高效的复盘）

## 关键提醒

- **总周期估算**：约 3~4 周（每天 2~3 小时）。时间紧时阶段 4 的代码生成器可砍
- **接口契约风险点**：登录接口的 `token` 字段名、`getRouters` 的返回结构必须与前端严格一致，否则页面白屏——这是全项目最容易踩的坑

## 进度

- [ ] 阶段 0：环境与工程搭建
- [ ] 阶段 1：common 通用层
- [ ] 阶段 2：认证 + RBAC 权限
- [ ] 阶段 3：通用业务模块
- [ ] 阶段 4：外围扩展
- [ ] 阶段 5：收尾
