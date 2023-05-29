## AOP

Nest 使用的是 MVC 架构。在 MVC 架构下，请求会先发送给 Controller，由它调度 Module 层的 Service 来完成业务逻辑，然后返回对应的 View。

![MVC](../assets/mvc.png)

在这个流程中，Nest 提供了 AOP 的能力。

Nest 在调用 Controller 之前和之后增加了执行通用逻辑的阶段。如果要在调用链路里加一段通用逻辑（比如：日志、权限、异常处理等），就可以在这个阶段去执行。

![AOP](../assets/aop.png)

这种增加的横向扩展点就叫做切面，透明的加入切面逻辑的编程方式就叫 AOP（面向切面编程）。

AOP 的好处是可以把一些通用逻辑分离到切面中，保持业务逻辑的存粹性，这样切面逻辑可以复用，还可以动态的增删。

### Nest 中的 AOP

Nest 中实现 AOP 的方式一共有五种：Middleware、Guard、Pipe、Interceptor、ExceptionFiler。

#### Middleware

Nest 的底层是 Express，自然也可以使用中间件，但是做了进一步的细分，分为全局中间件和路由中间件。

通过类定义的中间件必须实现 `NestMiddleware` 接口：

```typescript
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next();
  }
}

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {
  configure(consumer: MiddlewareConsumer) {
    // 全局使用
    consumer.apply(LoggerMiddleware).forRoutes('*');

    // 部分路由使用
    // consumer.apply(LoggerMiddleware).forRoutes('cats');
  }
}
```

通过函数也能定义中间件：

```typescript
import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  console.log('Request...');
  next();
}

// 全局使用
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(logger);
  await app.listen(3000);
}

// 部分路由使用
@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(logger).forRoutes('cats');
  }
}
```

#### Guard

Guard 是守卫的意思，可以用于在调用某个 Controller 之前判断权限，返回 true 或者 false 来决定是否放行。

Guard 必须实现 `CanActivate` 接口，守卫的 `canActivate` 方法中的 `context` 参数可以拿到请求信息：

```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable(boolean) {
    return true;
  }
}

// 全局使用
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalGuards(new RolesGuard());
  await app.listen(3000);
}

// 部分路由使用
@Controller()
@UseGuards(RolesGuard)
export class CatsController {
  constructor(private readonly catsService) {}

  getHello(): string {
    return this.catsService.getHello();
  }
}
```

#### Interceptor

Interceptor 是拦截器的意思，可以在目标 Controller 方法前后加入一些逻辑。

Interceptor 必须实现 `NestInterceptor` 接口，拦截器中的 `intercept` 方法中调用 `next.handle` 就会调用目标 Controller：

```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');

    const now = Date.now();
    return next.handle().pipe(tap(() => console.log(`After...${Date.now() - now}ms`)));
  }
}

// 全局使用
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalInterceptors(new LoggingInterceptor());
  await app.listen(3000);
}

// 部分路由使用
@Controller()
@UseInterceptors(LoggingInterceptor)
export class CatsController {
  constructor(private readonly catsService) {}

  getHello(): string {
    return this.catsService.getHello();
  }
}
```
