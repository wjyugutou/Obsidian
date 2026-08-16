第一次请求拿回响应头后：
1. 强缓存判断（优先看 Cache-Control，HTTP/1.1 标准）：
	1. max-age=31536000：1 年内不发请求，直接本地缓存（memory/disk cache）
	2. no-cache：注意！不是不缓存——意思是"每次都要向服务器验证" → 走协商
		1. 协商缓存：带着验证头发请求：
		2. If-None-Match:  ← ETag 优先（内容指纹，精确到字节）
		3. 服务器回 304 Not Modified → 用本地缓存；回 200 + 新内容 → 替换
	3. no-store：才是彻底不缓存
	4. immutable：配合 max-age，告诉浏览器期间绝对不用再验证

2. 优先级：Cache-Control > Expires（旧头）；ETag > Last-Modified（秒级精度，太弱）
3. 工程意义：webpack 给文件名加 contenthash——内容变 → 文件名变 → URL 变 → 强制请求新文件；配合 max-age 长缓存，旧文件永远命中本地缓存。这就是你天天看的 app.8f3k2.js 存在的意义。
 
## max-age: 3600 的资源过期后，浏览器发请求时具体带什么头？服务端 304 和 200 分别意味着什么？为什么 304 比 200 "便宜"？

请求: If-None-Match: "abc123"(ETag)

服务器对比自己现在的内容：
- 内容没变 → 返回 304 Not Modified——没有 body，只有几行响应头
（翻译：没新货，你手里那本还用着）
- 内容变了 → 返回 200，带上完整的新内容（几 MB 的 body）

