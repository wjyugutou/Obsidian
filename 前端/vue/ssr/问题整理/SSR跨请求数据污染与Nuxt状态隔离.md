# SSR 跨请求数据污染与 Nuxt 状态隔离

## 问题根源

模块级单例（`const cache = new Map()`）× 多用户共享进程

```
浏览器 A (用户A) ──┐
                   ├──► Node 进程 (单实例长期运行)
浏览器 B (用户B) ──┘        └─ 模块首次 import 时 new Map() 一次
                            └─ 所有请求共享这同一个 Map
```

- SPA 时代：每个用户一个浏览器 → 进程级隔离天然存在，模块级缓存安全
- SSR 时代：单进程多请求 → 所有请求共享同一模块变量 → **静默泄漏**

## 关键判断：数据是否用户私有

| 数据类型 | 例子 | 进程级缓存是否安全 |
|----------|------|-------------------|
| 全用户公共 | 字典、国家列表、全局配置 | ✅ 安全，反而省请求 |
| 用户私有 | 个人信息、权限、订单、租户隔离数据 | ❌ 必须 request 级隔离 |

> 一句话判断：**换个用户来看，这个缓存里的值还一样吗？**
> 一样 → 进程级 OK；不一样 → 必须隔离

## Nuxt 中的方案

### 1. 客户端要用的共享状态 → `useState`

Nuxt 内置的 SSR 友好版 `ref`，**每请求一份独立 payload**，序列化后随 HTML 注入给客户端 hydration：

```ts
const state = useState('key', () => initValue)
```

- 值从 `nuxtApp.payload.state` 存取
- 只能在组件 setup / plugin / route middleware 中调用（有 Nuxt 上下文）
- **不能在模块顶层（global/ambient context）调用**
- 自动请求隔离 → 天然安全

### 2. 纯服务端公共缓存 → `useStorage`

字典这类公共数据不值得每个请求重建：

```ts
const dictCache = useStorage('cache:dict')

export async function loadDict(name: string) {
  const cached = await dictCache.getItem(name)
  if (cached) return cached
  const res = await fetchDict(name)
  await dictCache.setItem(name, res)
  return res
}
```

- 可配 memory、Redis、FS 等驱动
- 比裸 `new Map()` 更可控：可观测、可失效、可跨实例共享

### 3. 纯服务端私有数据 → `event.context`

Nitro（Nuxt server 层）底层已用 `AsyncLocalStorage` 包裹每个请求：

```ts
export default defineNitroPlugin((nitroApp) => {
  nitroApp.hooks.hook('request', (event) => {
    event.context.dictCache = new Map()
  })
})
// 业务侧：event.context.dictCache.get(...)
```

### 决策表

| 数据性质 | 方案 | 隔离级别 |
|---------|------|---------|
| 跨 SSR/客户端共享、需要 hydration | `useState` | 请求级（payload 隔离） |
| 仅服务端的公共数据 | `useStorage` | 进程级（显式缓存层） |
| 仅服务端的私有数据 | `event.context` | 请求级（h3 context） |
| 纯客户端临时状态 | `ref` / `reactive` | 客户端「够用就好，不要滥用 `useState」 |

## useRef vs reactive vs useState 的选择

- `useState` 不是 `reactive` 的替代品，是 `ref` 的 SSR 友好版
- `useState` 的**额外成本**：值会被序列化进 HTML 的 `payload.state`，首屏 HTML 体积增加
- 判断顺序：是否需要 SSR hydration → 是否需要响应式 → 选合适的工具

## 模块级缓存的 SSR 陷阱

```ts
// ❌ SPA 时代合理，SSR 时代危险
const dictMapCache = new Map<string, DictObject>()
```

这个模式在 SSR 中的三个问题：
1. **跨请求污染**：用户 A 数据泄漏给用户 B
2. **请求间隔离不可见**：不报错、不崩溃，只有数据错了才被发现
3. **单元测试污染**：测试间共享状态，需要手动清理

### 优化方向

- 公共数据：改用 `useStorage` + 显式缓存层
- 跨 SSR/Client 数据：改用 `useState`，自动请求隔离
- 防污染：公共数据 `deepFreeze` + 缓存 Promise 而非结果（自动去重）
