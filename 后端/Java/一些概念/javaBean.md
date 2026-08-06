JavaBean 是 Java 中一种**约定俗成的组件规范**，不是关键字、不是注解、也不是接口——它是一套"长得像这样就能被框架识别"的**命名与结构契约**。
> [!note]
> JavaBean 之于 Java，就像 **"导出约定"** 之于 Node.js 生态。NestJS 用装饰器 `@Injectable()` 显式声明，JavaBean 用命名规范隐式声明。本质都是"让框架认识你"。


|规则|说明|
|:--|:--|
|1. 公共无参构造器|`public ClassName()` —— 框架反射实例化的入口|
|2. 私有字段 + getter/setter|字段封装，通过 `getXxx()` / `setXxx()` / `isXxx()` 访问|
|3. 可序列化|实现 `Serializable` 接口（可选但推荐）|
|4. 类是 public 且非抽象|可被外部实例化和继承|

### ⚠️ 2026 年的现实：JavaBean ≠ 现代 Java 最佳实践
|时代|做法|问题|
|:--|:--|:--|
|传统 JavaBean|无参 + getter/setter + 可变|对象创建后状态不确定，线程不安全|
|现代 Java|Record / Lombok `@Value` / 构造器注入|不可变、语义清晰、编译期安全|
#### 现在还在用 JavaBean 的场景

- ✅ **JPA Entity**（Hibernate 要求无参 + setter）
- ✅ **Spring MVC 表单绑定**（`@ModelAttribute` 依赖 setter）
- ✅ **老旧 XML 配置项目**
- ✅ **第三方库兼容**（Jackson/BeanUtils 等仍按 Bean 规范工作）

#### 不应该再用 JavaBean 的场景

- ❌ Service / Repository → 用构造器注入
- ❌ API Response DTO → 用 `record`
- ❌ Domain Value Object → 用不可变类
- ❌ 配置类 → 用 `@ConfigurationProperties` + record

> [!note]
> 遇到 "JavaBean" 这个词时：
> 
> ├── 在教程/老代码中 → 理解为"带 getter/setter 的 POJO"
> 
> ├── 在 JPA/表单绑定中 → 遵守规范，老实写无参+setter
> 
> └── 在新业务代码中 → 优先用 record / Lombok，别主动写 JavaBean
