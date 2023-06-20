## Fastify

Fastify 是一个高性能的 Web 服务框架，通常要比 Express 快两倍。Nest 默认使用了 Express， 原因是 Express 使用广泛，并且拥有丰富的第三方包，可供 Nest 开箱即用。

虽然默认使用了 Express，但 Nest 提供了一个框架适配器，可以轻松的切换底层框架。在一些需要高性能的场景时，Fastify 会是更好的选择。

### 安装

安装 Fastify：

```shell
pnpm add @nestjs/platform-fastify
```

### 适配器

安装 Fastify 后，就可以使用`FastifyAdapter`：

```typescript
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(AppModule, new FastifyAdapter());
  await app.listen(3000);
}

bootstrap();
```

当使用 `FastifyAdapter` 时，Nest 使用 Fastify 作为 HTTP 提供程序，这意味着依赖 Express 的配置可能不再有用，此时就要使用 Fastify 等效的包。

### 特定于库的方法

与使用 Express 时类似，Fastify 也可以通过`@Req()`和`@Res()`来注入请求和响应对象：

```typescript
import { FastifyRequest, FastifyReply } from 'fastify';

@Controller()
export class AppController {
  @Get()
  getHello(@Req() request: FastifyRequest, @Res() reply: FastifyReply) {
    reply.send('hello');
  }
}
```

使用 Fastify 的类型需要安装 `fastify`包。
