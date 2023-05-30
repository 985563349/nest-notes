## IOC

Inverse Of Control（IOC），即控制反转，是一种设计模式，用于实现代码之间的松耦合。

在传统的过程式编程中，代码通常会按照一定的顺序、在一定的时机进行调用，也就是控制权通常由开发者手动控制。而 IOC 模式则是将控制权交给了框架或容器，由框架或容器自动执行代码，代码对外部环境的依赖通过外部注入而不是自己创建，这样使代码之间的依赖更加灵活和可维护。

IOC 常见的实现方式包括依赖注入（DI）和控制反转容器（IOC 容器）。

### Nest 中的 IOC

#### Service

Service 类上声明了 `@Injectable` ，表明这个类是可注入的，那么 Nest 就会把它放到 IOC 容器中。

```typescript
@Injectable()
export class AppService {}
```

#### Controller

Controller 类上声明了 `@Controller` ，表明这个类是可以被注入的，Nest 也会把它放到 IOC 容器中。

```typescript
import { AppService } from './AppService';

@Controller()
export class AppController() {
  constructor(private readonly appService: AppService) {}
}
```

为什么 Controller 是单独的装饰器？

因为 Service 是可以被注入，也可以注入到别的对象的，所以用 `@Injectable` 声明。而 Controller 只需要被注入，所以 Nest 单独给它提供了 `@Controller` 装饰器。

#### Module

Module 类上声明了 `@Module` ，表面这个类是一个模块。Nest 使用模块来组织应用程序结构。

```typescript
import { AppController } from 'AppController';
import { AppService } from 'AppService';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

Nest 会从 AppModule（根模块）开始解析类上通过装饰器声明的依赖信息，自动创建和组装对象。所以 Controller 类中只是声明了对 Service 类的依赖就能获取到它。

Nest 的模块机制，可以把不同业务的 Controller、Service 等放到不同模块里。当引入别的模块后，引入模块导出的 provider 就可以在当前模块内进行注入。

```typescript
// OtherModule.ts
@Module({
  imports: [],
  controllers: [OtherController],
  providers: [OtherService],
  exports: [OtherService],
})
export class OtherModule {}

// AppController.ts
@Module({
  imports: [OtherModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

示例中的 AppModule 内部可以使用 OtherModule 导出的 OtherService 来进行注入。

### 为什么需要 IOC

Nest 这套 IOC 机制看似繁琐，但却解决了后端系统中对象依赖关系错综复杂的问题。

通常在后端系统中会有很多对象：

- Controller：接收 HTTP 请求，调用 Service，返回响应。
- Service：实现业务逻辑。
- Repository：实现数据操作。

此外还有数据库连接对象`DataSource`，配置对象`Config`等等。

这些对象往往有着错综复杂的关系，比如：Controller 依赖 Service 实现业务逻辑，Service 依赖 Repository 来做数据库操作，Repository 依赖 DataSource 来建立数据库连接，DataSource 又需要从 Config 中拿到用户名密码等信息。

这就导致了创建这些对象是很复杂的，要理清它们之间的依赖关系，哪个先创建，哪个后创建。而且这些对象通常并不需要每次都 new 一个新的，也就是要保持单例。

这是后端系统都有的痛点问题，IOC 的出现就是为了解决这个问题。

#### IOC 的实现思路

IOC 有一个放置对象的容器，程序初始化时会扫描类上声明的依赖关系，然后把这些类都 new 一个实例放到容器里。创建实例时还会将依赖项注入进去。

这种依赖注入的方式叫做：Dependency Injection，简称 DI。

由手动创建对象，然后组装，到声明依赖，等待被注入。这就是 Inverse Of Control，控制反转。
