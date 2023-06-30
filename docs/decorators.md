## 装饰器

Nest 是围绕一种称为装饰器的语言功能构建的。

### 内置装饰器

#### @Module

声明模块。

```typescript
@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

#### @Controller

声明控制器。

```typescript
@Controller()
export class AppController {}
```

#### @Injectable

声明依赖消费者，被声明的类可以自动注入依赖。

```typescript
@Injectable()
export class AppService {}
```

#### @Inject

手动注入依赖。

```typescript
// 属性注入
@Controller()
export class AppController {
  @Inject(AppService)
  private readonly appService: AppService;
}

// 构造器注入
@Controller()
export class AppController {
  constructor(@Inject('User') user: { name: string }) {}
}
```

#### @Optional

声明依赖为可选，默认情况下声明的依赖容器中不存在，创建对象时会报错。

```typescript
@Controller()
export class AppController {
  constructor(@Optional() private readonly appService: AppService) {}
}
```

#### @Catch

为异常过滤器指定捕获的异常类型，未指定默认捕获所有异常类型。

```typescript
@Catch(HttpException)
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {}
}
```

#### @UseFilters

应用异常过滤器。

```typescript
@Controller()
export class AppController {
  @Get()
  @UseFilters(AllExceptionsFilter)
  getHello() {
    throw new HttpException('error', HttpStatus.BAD_REQUEST);
  }
}
```

#### @UseGuards

应用守卫。

```typescript
@Controller()
@UseGuards(RolesGuard)
export class AppController {}
```

#### @UseInterceptors

应用拦截器。

```typescript
@Controller()
@UseInterceptors(LoggerInterceptor)
export class AppController {}
```

#### @UsePipes

应用管道。

```typescript
@Controller()
export class AppController {
  @Get(':id')
  @UsePipes(ValidationPipe)
  findOne(@Param('id') id: number) {}
}
```

#### @Param

获取路径参数。

```typescript
@Get('id')
findOne(@Param('id') id: string) {}
```

#### @Query

获取查询字符串。

```typescript
@Get()
find(@Query('name') name: string, @Query('age', ParseIntPipe) age: number) {}
```

#### @Body

获取请求体数据，`@Body` 能将请求体数据实例化成一个 dto 对象。

```typescript
@Post()
create(@Body() createCatDto: CreateCatDto) {}
```

#### 请求处理

`@Get` 用于处理 GET 请求，`@Post` 用于处理 POST 请求。此外还可以使用`@Put`、`@Delete`、`@Patch`、`@Options`、`@Head`装饰器处理其他类型的请求。

```typescript
@Get()
findAll(@Query() paginationQuery) {}

@Post()
create(@Body() createCatDto: CreateCatDto) {}
```

#### @SetMetadata

设置元数据。

```typescript
@Controller()
@SetMetadata('roles', ['user'])
export class AppController {
  @Get()
  @UseGuards(RolesGuard)
  @SetMetadata('roles', ['admin'])
  findAll() {}
}
```

指定的元数据可以在 `Guard` 或者 `Interceptor` 中获取。

```typescript
export class RolesGuard {
  @Inject(Reflector)
  private readonly reflector: Reflector;

  canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
    const classMetadata = this.reflector.get('roles', context.getClass());
    const handlerMetadata = this.reflector.get('roles', context.getHandler());
    return true;
  }
}
```

#### @Headers

获取请求头。

```typescript
@Get()
find(@Headers('Accept') accept: string, @Headers() headers: Record<string, any>) {}
```

#### @Ip

获取请求 IP。

```typescript
@Get()
find(@Ip() ip: string) {}
```

#### @Session

获取 Session

```typescript
@Get()
find(@Session() session) {}
```

#### @HostParam

获取 host。

```typescript
@Get()
find(@HostParam() host) {}
```

#### @Req

获取 request 对象，`@Request` 是它的别名。

```typescript
@Get()
find(@Req() req: Request) {}
```

#### @Res

获取 response 对象，`@Response` 是它的别名。注入 response 对象后，Nest 不会再把 handler 的返回值作为响应内容，可以手动返回响应。

```typescript
@Get()
find(@Res() response: Response) {
  res.end('Hello World!');
}
```

也可以通过 `passthrough` 参数告诉 Nest 依旧把 handler 的返回值作为响应内容。

```typescript
@Get()
find(@Res({ passthrough: true }) response: Response) {
  return 'Hello World!';
}
```

#### @Next

当有两个 handler 处理同一个路由时，可以在第一个 handler 里注入 `NextFunction`，调用它来把请求转发到第二个 handler。Nest 不会处理注入 `NextFunction` 的 handler 的返回值。

```typescript
@Get('handler')
handler(@Next() next: NextFunction) {
  console.log('handler1')
  next()
  return 'Hello Next.JS!'
}

@Get('handler')
handler2() {
  return 'Hello World!'
}
```

#### @HttpCode

修改 http 状态码。

```typescript
@Get()
@HttpCode(222)
find() {}
```

#### @Header

设置响应头。

```typescript
@Get()
@Header('aaa', 'bbb')
find() {}
```

#### @Redirect

重定向路由。

```typescript
@Get()
@Redirect('https://www.github.com')
find() {}
```

#### @Global

定义全局模块。

```typescript
@Global()
@Module({
  controllers: [GlobalController],
  providers: [GlobalService],
  exports: [GlobalService],
})
export class GlobalModule {}
```

### 自定义装饰器

装饰器本质上就是一个函数，这个函数可以在内部修改类的行为。

```typescript
// 可传递参数的装饰器本质上是一个高阶函数。
const Roles = (...roles: string[]) => SetMetadata('roles', roles);

@Controller()
export class AppController {
  @Get()
  @Roles('admin')
  @UseGuards(RolesGuard)
  findAll() {}
}
```

#### 参数装饰器

Nest 提供了一个辅助方法 `createParamDecorator` 用来创建参数装饰器。

```typescript
export const User = createParamDecorator((data: unknown, ctx: ExecutionContext) => {
  const request = ctx.switchToHttp().getRequest();
  return request.user;
});

@Get()
findOne(@User() user: UserEntity) {}
```

自定义参数装饰器可以传递参数。

```typescript
export const User = createParamDecorator((data: string, ctx: ExecutionContext) => {
  const request = ctx.switchToHttp().getRequest();
  const user = request.user;
  return data ? user?.[data] : user;
});

@Get()
findOne(@User('firstName') firstName: string) {}
```

也可以使用管道。

```typescript
@Get()
findOne(
  @User(new ValidationPipe({ validateCustomDecorators: true }))
  user: UserEntity,
  @User('age', new ParseIntPipe())
  age: number
) {}
```

自定义装饰器的首个参数如果是 Pipe，那么它不会被 `createParamDecorator` 的回调参数接收（内部对参数的类型做了判断）。

`ValidationPipe` 默认情况下不验证自定义参数装饰器，开启验证需要设置 `validateCustomDecorators` 选项为 `true` 。

#### 装饰器组合

组合多个装饰器，Nest 同样也提供了辅助方法 `applyDecorators` 。

```typescript
export function Auth(...roles: Role[]) {
  return applyDecorators(
    SetMetadata('roles', roles),
    UseGuards(AuthGuard, RolesGuard),
    ApiBearerAuth(),
    ApiUnauthorizedResponse({ description: 'Unauthorized' })
  );
}
```

使用组合装饰器，就有通过单个声明应用四个装饰器的效果。

```typescript
@Get('users')
@Auth('admin')
findAllUsers() {}
```
