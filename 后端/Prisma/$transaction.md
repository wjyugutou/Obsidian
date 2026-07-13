是 Prisma Client 提供的**数据库事务**方法，用于确保一组数据库操作要么**全部成功提交**，要么**全部回滚**，保证数据一致性。

### 回调函数用法
```ts
const result = await this.prisma.$transaction(async (tx) => {
  // tx 是一个事务内的 PrismaClient 代理
  const user = await tx.user.create({
    data: { name: 'Alice', email: 'alice@example.com' },
  });

  await tx.profile.create({
    data: { userId: user.id, bio: 'Hello' },
  });

  await tx.order.create({
    data: { userId: user.id, amount: 99.9 },
  });

  // 如果这里抛出异常，上面三个操作全部回滚
  return user;
}, { 

		maxWait: 5000,         // 获取锁的最大等待时间(ms)，默认 2000
		timeout: 10000,        // 事务最大执行时间(ms)，默认 5000
		isolationLevel: 'ReadCommitted', // 隔离级别

});
```

### 数组参数用法
```ts
const [user, post] = await this.prisma.$transaction([
  this.prisma.user.create({ data: { name: 'Alice', email: 'a@b.com' } }),
  this.prisma.post.create({ data: { title: 'Hello', authorId: 1 } }),
]);
```

### 两种模式对比

| 特性 | 交互式 `$transaction(fn)` | 批量 `$transaction([...])` |
|------|--------------------------|---------------------------|  
| 操作间依赖 | ✅ 支持 | ❌ 不支持 |  
| 条件逻辑 | ✅ 支持 | ❌ 不支持 |  
| 外部调用(API/Redis) | ✅ 支持 | ❌ 无意义 |  
| 错误处理 | ✅ try/catch | ❌ 只能整体失败 |  
| 性能 | 略低（串行+锁） | 略高（并行批处理） |  
| 适用场景 | 复杂业务逻辑 | 简单并行写入 |

### ⚠️ 注意事项

1. **不要在事务内做耗时操作**（HTTP请求、文件上传、发送邮件），会长时间持有数据库连接
2. **NestJS 11 + Zod**：事务不影响 DTO 验证，验证仍在 Controller/Guard 层完成
3. **MongoDB 限制**：需要 Replica Set 或 Sharded Cluster 才支持事务
4. **SQLite 限制**：不支持并发事务，同一时间只有一个事务能执行
5. **嵌套事务**：Prisma 不支持真正的嵌套事务，内层 `$transaction` 会被忽略，直接使用外层事务

