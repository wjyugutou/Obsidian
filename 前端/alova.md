```ts
import type { ApiResponse, RequestSuccessCallbackResult } from './tools/enum'
import AdapterUniapp from '@alova/adapter-uniapp'
import { createAlova } from 'alova'
import vueHook from 'alova/vue'
import { ContentTypeEnum, ResultEnum, ShowMessage } from './tools/enum'

export const baseURL = getEnvBaseUrl()

export const alovaInstance = createAlova({
  baseURL,
  ...AdapterUniapp(),
  statesHook: vueHook,
  beforeRequest: (method) => {
    method.config.headers = {
      ContentType: ContentTypeEnum.JSON,
      Accept: 'application/json, text/plain, */*',
      ...method.config.headers,
    }

    if (method.meta?.ignoreAuth !== true) {
      const { token } = useToken()

      method.config.headers.Authorization = `Bearer ${token}`
    }
  },
  responded: (response, method) => {
    const { config } = method
    const { requestType } = config

    const { error } = useGlobalNotify()
    const { statusCode, data } = response as RequestSuccessCallbackResult

    if (requestType === 'upload' || requestType === 'download') {
      return response
    }

    if (statusCode !== 200) {
      const errorMessage = ShowMessage(statusCode) || `HTTP请求错误[${statusCode}]`
      error(`请求失败, ${errorMessage}`)
      throw new Error(`${errorMessage}`)
    }

    const { code, msg, data: rawData } = data as ApiResponse
    if (code !== ResultEnum.Success0 && code !== ResultEnum.Success200) {
      if (code === ResultEnum.Unauthorized && useCurrentPath() !== import.meta.env.VITE_LOGIN_URL) {
        uni.reLaunch({ url: import.meta.env.VITE_LOGIN_URL })
      }
      if (config.meta?.hideNotify !== true) {
        error(`${data.msg}`)
      }
      throw new Error(`请求失败 ${code}, ${data.msg}`)
    }

    return (method.meta?.originalRes ? data : rawData) as ApiResponse
  },
  timeout: 20000,
  cacheFor: null,
})

export default alovaInstance

```