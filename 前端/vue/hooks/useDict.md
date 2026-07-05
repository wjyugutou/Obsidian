在SSR中应注意 [[SSR跨请求数据污染与Nuxt状态隔离]]
```ts
import type { ToRefs } from 'vue'
// import type { DictResult } from '@/api'
import { getDictByTypeApi } from '@/api'

interface DictResult {
  id: string
  dictValue: string
  dictLabel: string
  [key: string]: any
}

export interface DictObject {
  list: { value: DictResult['dictValue'], label: DictResult['dictLabel'], [key: string]: any }[]
  /** dictValue -> dictLabel */
  valueToLabel: Record<string, string>
  /** dictLabel -> dictValue */
  labelToValue: Record<string, DictResult['dictValue']>
}

type Dict<T extends string[]> = {
  [K in T[number]]: DictObject
}

// 缓存 Promise 而非结果：避免多个组件同时首次请求同一字典时发起重复请求。
// 模块级单例，若启用 SSR 需改为 request 级隔离，避免跨用户数据污染。
const dictPromiseCache = new Map<string, Promise<DictObject>>()

function loadDict(dict: string): Promise<DictObject> {
  const cached = dictPromiseCache.get(dict)
  if (cached) return cached
 
  const promise = getDictByTypeApi(dict)
    .then((res: DictResult[]) => deepFreeze(transformDict(res)))
    .catch((err: unknown) => {
      // 失败移除缓存，允许后续重试
      dictPromiseCache.delete(dict)
      throw err
    })

  dictPromiseCache.set(dict, promise)
  return promise
}

function transformDict(items: DictResult[]): DictObject {
  const valueToLabel: Record<string, string> = {}
  const labelToValue: Record<string, DictResult['dictValue']> = {}
  for (const item of items) {
    valueToLabel[item.dictValue] = item.dictLabel
    labelToValue[item.dictLabel] = item.dictValue
  }
  return {
    list: items.map(item => ({ ...item, value: item.dictValue, label: item.dictLabel })),
    valueToLabel,
    labelToValue,
  }
}

// 字典为只读共享数据，深冻结防止某个组件修改污染全局缓存
function deepFreeze<T>(obj: T): T {
  if (obj && typeof obj === 'object') {
    Object.values(obj as Record<string, unknown>).forEach(deepFreeze)
    Object.freeze(obj)
  }
  return obj
}

/**
 * 字典hooks
 * @param dicts 字典值
 */
export function useDict<T extends string[]>(...dicts: T): ToRefs<Dict<T>> {
  const initvalue = dicts.reduce((prev, item) => {
    prev[item as T[number]] = { list: [], valueToLabel: {}, labelToValue: {} }
    return prev
  }, {} as Dict<T>)

  const dictMap = reactive(initvalue) as Dict<T>

  dicts.forEach((dict) => {
    loadDict(dict)
      .then(obj => { dictMap[dict as T[number]] = obj })
      .catch(err => {
        // 保持空值不阻断渲染；接入错误上报可在此扩展
        console.warn(`[useDict] 加载字典 "${dict}" 失败:`, err)
      })
  })

  return toRefs(dictMap)
}

```