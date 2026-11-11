```ts
import type { AxiosRequestConfig, AxiosResponse, InternalAxiosRequestConfig } from 'axios'
import type { ApiResponse } from './tools/enum'
import axios from 'axios'
import router from '@/router'

type Result<T = any, B = object> = {
  code: number
  msg: string
  data: T
  [key: string]: any
} & B

interface CustomConfig {
  meta?: {
    /** 是否需要设置token */
    isToken?: boolean
    /** 是否需要显示错误提示 */
    showMsg?: boolean
    // loading?: boolean
    // headers?: Record<string, any>
  }
}

type RequestConfig = AxiosRequestConfig & CustomConfig

type InterceptorsConfig = InternalAxiosRequestConfig & CustomConfig

interface InterceptorsResponse extends AxiosResponse {
  config: InterceptorsConfig
}

export const baseURL = import.meta.env.BASE_URL

export const axiosInstance = axios.create({
  baseURL, // url = base url + request url
  timeout: 15000, // request timeout
  headers: {
    'Content-Type': 'application/json',
  },
})

axiosInstance.interceptors.request.use(
  (config: InterceptorsConfig) => {
    // 是否需要设置 token
    const isToken = config.isToken !== false

    if (isToken) {
      const { token } = useToken()

      if (token.value) {
        config.headers.Authorization = `Bearer ${token.value}` // 让每个请求携带自定义token 请根据实际情况自行修改
      }
    }

    return config
  },
  (error) => {
    console.error(error)

    throw error
  },
)

function showFailToast(message: string) {
  alert(message)
}

axiosInstance.interceptors.response.use(
  (response: InterceptorsResponse) => {
    const showMsg = response.config.meta?.showMsg !== false

    const res = response.data as ApiResponse

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

export default function request<T = unknown>(config: RequestConfig) {
  return axiosInstance.request<unknown, T>(config)
}


```