```ts
import type { AxiosRequestConfig, AxiosResponse, InternalAxiosRequestConfig } from 'axios'
import router from '@/router'
import axios from 'axios'

type Result<T = any, B = object> = {
  code: number
  msg: string
  data: T
  [key: string]: any
} & B

interface CustomConfig {
  /** 是否需要设置token */
  isToken?: boolean
  /** 是否需要显示错误提示 */
  showMsg?: boolean
  // loading?: boolean
  // headers?: Record<string, any>
}

type RequestConfig = AxiosRequestConfig & CustomConfig

type InterceptorsConfig = InternalAxiosRequestConfig & CustomConfig

interface InterceptorsResponse extends AxiosResponse {
  config: InterceptorsConfig
}

export const baseURL = import.meta.env.PUBLIC_BASE_API

const request = axios.create({
  baseURL, // url = base url + request url
  timeout: 15000, // request timeout
  headers: {
    'Content-Type': 'application/json',
  },
})

request.interceptors.request.use(
  (config: InterceptorsConfig) => {
    // 是否需要设置 token
    const isToken = config.isToken !== false

    if (isToken) {
      const token = useToken().access_token

      if (token) {
        config.headers.Authorization = `Bearer ${token}` // 让每个请求携带自定义token 请根据实际情况自行修改
      }
    }

    return config
  },
  (error) => {
    console.error(error)

    throw error
  },
)

request.interceptors.response.use(
  (response: InterceptorsResponse) => {
    const showMsg = response.config.showMsg === undefined ? true : response.config.showMsg

    const res = response.data

    if (res instanceof Blob) {
      return response.data
    }
    else if (res.code === 200) {
      return res.data
    }
    else {
      showMsg && showFailToast(res.msg || '网络错误，请重试')

      if (res.code === 401) {
        router.push('/login')
        return null
      }
      return null
    }
  },
  (error) => {
    console.error({ error })

    showFailToast(`${error.message} 服务器错误，请重试`)
    return null
  },
)

export function get<T = unknown, B = object>(url: string, config?: RequestConfig): Promise<Result<T, B>> {
  return request.get(url, config)
}

export function post<T = unknown, B = object>(url: string, data?: any, config?: RequestConfig): Promise<Result<T, B>> {
  return request.post(url, data, config)
}

export function put<T = unknown, B = object>(url: string, data?: any, config?: RequestConfig): Promise<Result<T, B>> {
  return request.put(url, data, config)
}

export function del<T = unknown, B = object>(url: string, config?: RequestConfig): Promise<Result<T, B>> {
  return request.delete(url, config)
}

export default request

```