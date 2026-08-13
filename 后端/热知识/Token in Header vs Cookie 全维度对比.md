## 传输与存储机制

| 维度   | Bearer Header（token）              | httpOnly Cookie                                        |
| ---- | --------------------------------- | ------------------------------------------------------ |
| 存哪   | 前端自管（localStorage / memory）       | 浏览器自动存，JS 不可读                                          |
| 发送方式 | 前端手动加 `Authorization: Bearer xxx` | 浏览器自动带（同源 / 配 SameSite）                                |
| 跨域   | 天然支持，header 跨域简单                  | 要 `SameSite=None; Secure`（仅 HTTPS）+ CORS `credentials` |
| 移动端  | 通用（App/小程序/SSR 都能用）               | 锁死浏览器场景                                                |

## 安全攻击面对比

| 攻击 | Bearer Header | httpOnly Cookie | 谁赢 |
|---|---|---|---|
| XSS 偷 token | ❌ 能偷走（`localStorage.getItem`） | ✅ JS 读不到 | Cookie 赢 |
| XSS 滥用会话 | ❌ 能借用户身份发请求 | ❌ 同样能借用户身份发请求 | 平（cookie 没赢，XSS 仍可借会话） |
| CSRF | ✅ 天然免疫（不带 cookie） | ❌ 要靠 SameSite / CSRF token 防护 | Bearer 赢 |
| token 泄露利用 | 偷到即可用 | 偷到 cookie 即用 | 平 |

**关键洞察**: cookie 防 XSS 偷 token，但防不住 XSS 滥用会话。

XSS 直接：

```js
fetch('/api/transfer', {
  credentials: 'include'
})