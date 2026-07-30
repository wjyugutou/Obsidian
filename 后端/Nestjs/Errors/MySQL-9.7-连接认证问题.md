---
updated: 2026-07-17
---
> 环境：NestJS 11 + Prisma 7 + `@prisma/adapter-mariadb@7.8.0` + MySQL 9.7（Windows）

## 问题 1：pool timeout（数据库连接超时）

### 现象

请求任意数据库接口时报错：

```
pool timeout: failed to retrieve a connection from pool after 10013ms
(pool connections: active=0 idle=0 limit=10)
```

`active=0 idle=0` —— 连接池一个连接都没建立成功。

### 排查过程

| 检查项 | 结果 |
|---|---|
| MySQL 服务是否运行 | ✅ `mysqld` 进程存在，端口 3306 监听中 |
| TCP 连通性（Node `net.connect`） | ✅ 可连接 |
| MySQL CLI 连接 | ✅ `mysql -u root -p` 正常 |
| `mariadb` npm 包连接 | ❌ 同样超时 |
| 加 `allowPublicKeyRetrieval: true` | ✅ 连接成功 |

### 根因

MySQL 9.7 默认认证插件为 `caching_sha2_password`：

```sql
SELECT user, host, plugin FROM mysql.user WHERE user='root';
-- root  localhost  caching_sha2_password
```

`mariadb` npm 包（v3.4.5）在协商 `caching_sha2_password` 时，需要从服务端获取公钥来加密密码。默认出于安全考虑不允许明文获取公钥，必须显式开启 `allowPublicKeyRetrieval: true`。

### 修复

```ts
// src/prisma/prisma.service.ts
adapter: new PrismaMariaDb({
  host: configService.get<string>('DB_HOST')!,
  port: configService.get<number>('DB_PORT')!,
  user: configService.get<string>('DB_USERNAME')!,
  password: configService.get<string>('DB_PASSWORD')!,
  database: configService.get<string>('DB_DATABASE')!,
  charset: configService.get<string>('database.charset') ?? 'utf8mb4',
  allowPublicKeyRetrieval: true, // ← 关键
}),
```

---

## 问题 2：`rejectUnauthorized` 不在类型 `PoolConfig` 中

### 现象

尝试升级到 SSL 连接时，TypeScript 报错：

```
"rejectUnauthorized"不在类型"PoolConfig"中
```

### 根因

`mariadb` 的类型定义（`node_modules/mariadb/types/share.d.ts:682`）：

```ts
ssl?: boolean | (SecureContextOptions & { rejectUnauthorized?: boolean });
```

`rejectUnauthorized` 是 **`ssl` 对象的属性**，不是 `PoolConfig` 的顶层属性。

### 错误写法

```ts
// ❌ rejectUnauthorized 放在顶层
new PrismaMariaDb({
  rejectUnauthorized: true,
})
```

### 正确写法

```ts
// 开发环境：启用 SSL 但不验证服务器证书
new PrismaMariaDb({
  ssl: true,
})

// 生产环境：验证 CA 证书
new PrismaMariaDb({
  ssl: {
    ca: readFileSync('/path/to/ca.pem'),
    rejectUnauthorized: true,
  },
})
```

---

## SSL vs allowPublicKeyRetrieval 的关系

```
无 SSL + caching_sha2_password → 需要 allowPublicKeyRetrieval: true
有 SSL + caching_sha2_password → SSL 已提供安全信道，不需要 public key retrieval
```

启用 SSL 后可以删掉 `allowPublicKeyRetrieval`。

### 三种方案的对比

| 方案 | 代码 | 安全性 | 适用场景 |
|---|---|---|---|
| 无 SSL | `allowPublicKeyRetrieval: true` | 低（密码用服务端公钥加密传输） | 开发 |
| SSL 跳过验证 | `ssl: true` | 中（加密传输，不验证服务端身份） | 开发 / 内网 |
| SSL + CA 验证 | `ssl: { ca: ..., rejectUnauthorized: true }` | 高 | 生产 |

---

## 当前项目方案

学习仓库，本地开发，采用最简方案：`allowPublicKeyRetrieval: true`。

未来生产化的路径：
1. 启用 SSL（`ssl: true` 或 `ssl: { ca, rejectUnauthorized: true }`）
2. 删除 `allowPublicKeyRetrieval: true`
3. CA 证书路径放 `.env`（`SSL_CA=/path/to/ca.pem`）

---

## 参考

- mariadb connector SSL 文档：https://github.com/mariadb-corporation/mariadb-connector-nodejs/blob/master/documentation/connection-options.md#ssl
- MySQL `caching_sha2_password` 插件：https://dev.mysql.com/doc/refman/9.7/en/caching-sha2-pluggable-authentication.html
]