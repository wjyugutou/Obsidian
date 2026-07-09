---
tags:
  - nestjs
  - backend
  - dependency-injection
  - architecture
  - 结构
created: 2026-07-09
---
## 📌 核心定义速查

| 属性 | 一句话定义 | 作用域 | 类比 |
| :--- | :--- | :--- | :--- |
| `providers` | 本模块**拥有**的服务 | 仅本模块内部可用（除非被 export） | 私有成员 + 公有成员 |
| `exports` | 本模块**愿意分享**的服务 | 对其他导入本模块的模块可见 | `public` 关键字 |
| `imports` | 本模块**需要依赖**的其他模块 | 引入其他模块已导出的 Provider | `import` 语句 |

> [!IMPORTANT] 黄金法则
> - **`providers`** = "我有什么"
> - **`exports`** = "我给别人什么"
> - **`imports`** = "我要别人什么"
> 
> 三者配合构成完整的 IoC 模块化体系，缺一不可。

---

## 🔍 详细说明

### providers
- 声明当前模块创建的 Provider（Service、Repository、Factory、Value 等）
- **默认私有**：未放入 `exports` 的 Provider 外部模块无法注入
- Controller **不需要**也不应该放在 `exports` 中

### exports
- 控制模块的**公共 API 边界**
- 支持三种导出形式：
    1. 直接导出 Provider：`exports: [CatsService]`
    2. 重新导出整个模块：`exports: [CommonModule]`（透传其所有已导出 Provider）
    3. 自定义 Token 导出：`exports: [{ provide: 'CONFIG', useValue: {...} }]`
- 只有被 export 的 Provider 才能被其他模块通过 DI 注入

### imports
- 将其他模块的**已导出 Provider** 纳入当前模块的 DI 容器
- 导入后无需再次在 `providers` 中声明同名类
- 支持 `forwardRef()` 解决循环依赖

---

## ⚠️ 常见反模式：为什么不要重复声明？

> [!DANGER] 错误做法
> 在多个模块的 `providers` 中重复声明同一个类

```typescript
// ❌ CatsModule 和 DogsModule 各自声明 CatsService
@Module({ providers: [CatsService] }) export class CatsModule {}
@Module({ providers: [CatsService] }) export class DogsModule {}