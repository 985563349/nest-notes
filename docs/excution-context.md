## 执行上下文

Nest 支持创建 HTTP 服务、WebSocket 服务，还有基于 TCP 通信的微服务。这些不同类型的服务都需要 Guard、Interceptor、Exception Filter 功能。但不同类型的服务能拿到的参数是不同的。比如 HTTP 服务可以拿到 request、response 对象，而 WebSocket 服务就没有。

如何让 Guard、Interceptor、Exception Filter 跨多种上下文复用，Nest 的解决方法是 `ArgumentsHost` 和 `ExecutionContext` 类。

### ArgumentsHost

`ArgumentsHost` 类提供了传递给处理程序的参数。它允许选择合适的上下文来获取参数。Nest 提供了 `ArgumentsHost` 的实例给需要获取的地方。例如，异常过滤器中的 `catch` 方法。

`getType` 方法可以用来确定当前运行的应用程序类型。

```typescript
@Catch()
export class AllExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    if (host.getType() === 'http') {
      // http context
    } else if (host.getType() === 'rpc') {
      // rpc context
    } else if (host.getType<GqlContextType>() === 'graphql') {
      // graphql context
    }
  }
}
```

`getArgs` 方法可以获取当前上下文参数。不同的服务类型下，可以获取到不同的参数。

例如，在 `HTTP` 服务下，可以获取到 `req`、`res` 和 `next`。

```typescript
const [req, res, next] = host.getArgs();
```

但 `getArgs` 会使应用程序与特定上下文耦合，更推荐的方法是使用应用方法来选择合适的应用上下文。

```typescript
// HttpArgumentsHost
host.switchToHttp();

// WsArgumentsHost
host.switchToWs();

// RpcArgumentsHost
host.switchToRpc();
```

### ExecutionContext

`ExecutionContext` 继承自 `ArgumentsHost`，提供了当前运行过程的额外信息。

![ExecutionContext](../assets/excution-context.png)

Nest 提供了 `ExecutionContext` 的实例给需要获取的地方。例如，守卫中的 `canActivate` 方法和拦截器中的 `intercept` 方法。

```typescript
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
    const ctx = context.switchToHttp();
    return true;
  }
}

export class LoggingInterceptor implements NestInterceptor {
  interceptor(context: ExecutionContext, next: CallHandler): Observable<any> {
    const ctx = context.switchToHttp();
    return next.handle();
  }
}
```

`ExecutionContext` 扩展的 `getClass` 和 `getHandler` 方法可以获取到当前的 class 和 handler。这为守卫和拦截器提供了极高的灵活性。最重要的是，可以从内部访问到控制器上设置的自定义元数据。

```typescript
// 在控制器中直接实用SetMetadata并不是好的做法，推荐创建一个自定义装饰器。
const Roles = (...roles: string[]) => @SetMetadata('roles', roles)

@Controller()
@Roles('user')
@UseGuards(RolesGuard)
export class AppController {
  @Get()
  @Roles('admin')
  getHello() {}
}

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private readonly reflector: Reflector){}

  canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
    const classMetadata = this.reflector.get('roles', context.getClass());
    const handlerMetadata = this.reflector.get('roles', context.getHandler());

    // 优先从 handler 中获取元数据，若 handler 上不存在，使用 class 上的元数据进行覆盖。
    const roles = this.reflector.getAllAndOverride('roles', [context.getHandler(), context.getClass()]); // => ['admin']
    // 合并 handler 和 class 上的元数据
    const roles2 = this.reflector.getAllAndMerge('roles', [context.getHandler(), context.getClass()]); // => ['user', 'admin']
  }
}
```
