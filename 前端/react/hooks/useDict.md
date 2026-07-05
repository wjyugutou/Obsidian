```ts
import { useEffect, useSyncExternalStore } from 'react'

import { getDictByTypeApi } from '@/api'

export interface DictObject {
  list: { value: string, label: string, [key: string]: any }[]
  map: Record<string, string | number | boolean>
}

interface DictResult {
  dictLabel: string
  dictValue: string
}

// useSyncExternalStore 要求 getSnapshot 返回稳定引用，否则无限渲染
const EMPTY_DICT: DictObject = { list: [], map: {} }

const dictMapCache = new Map<string, DictObject>()
const dictListeners = new Map<string, Set<() => void>>()
// 请求单飞：并发挂载复用同一个 Promise，避免重复请求
const dictPromiseCache = new Map<string, Promise<DictObject>>()

function subscribeDict(dict: string, listener: () => void): () => void {
  let listeners = dictListeners.get(dict)
  if (!listeners) {
    listeners = new Set()
    dictListeners.set(dict, listeners)
  }
  listeners.add(listener)
  return () => {
    listeners!.delete(listener)
    if (listeners!.size === 0)
      dictListeners.delete(dict)
  }
}

function getDict(dict: string): DictObject {
  return dictMapCache.get(dict) ?? EMPTY_DICT
}

function fetchDict(dict: string): Promise<DictObject> {
  const cached = dictMapCache.get(dict)
  if (cached)
    return Promise.resolve(cached)

  const flying = dictPromiseCache.get(dict)
  if (flying)
    return flying

  const promise = getDictByTypeApi(dict)
    .then((res: DictResult[]) => {
      const value = transformDict(res)
      dictMapCache.set(dict, value)
      dictPromiseCache.delete(dict)
      dictListeners.get(dict)?.forEach(cb => cb())
      return value
    })
    .catch((err) => {
      // 失败时清除 promise，允许后续调用重试
      dictPromiseCache.delete(dict)
      throw err
    })

  dictPromiseCache.set(dict, promise)
  return promise
}

export function useDictItem(dict: string): DictObject {
  const value = useSyncExternalStore(
    cb => subscribeDict(dict, cb),
    () => getDict(dict),
  )

  useEffect(() => {
    if (!dictMapCache.has(dict) && !dictPromiseCache.has(dict))
      fetchDict(dict)
  }, [dict])

  return value
}

function transformDict(dictValue: DictResult[]): DictObject {
  return {
    list: dictValue.map(item => ({ ...item, value: item.dictValue, label: item.dictLabel })),
    map: dictValue.reduce<DictObject['map']>((prev, item) => {
      prev[item.dictValue] = item.dictLabel
      prev[item.dictLabel] = item.dictValue
      return prev
    }, {}),
  }
}

```