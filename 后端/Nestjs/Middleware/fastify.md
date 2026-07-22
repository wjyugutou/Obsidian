中间件函数获取的是原始的 `req` 和 `res` 对象，而非 Fastify 的封装对象。这是底层使用的 `middie` 包以及 `fastify` 的工作机制 - 更多信息请参阅此[页面](https://www.fastify.io/docs/latest/Reference/Middleware/)
```ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { FastifyRequest, FastifyReply } from 'fastify';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
 use(req: FastifyRequest['raw'], res: FastifyReply['raw'], next: () => void) {
   console.log('Request...');
   next();
 }
}
```
