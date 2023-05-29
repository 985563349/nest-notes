## IOC

Inverse Of Control（IOC），即控制反转，是一种设计模式，用于实现代码之间的松耦合。在传统的过程式编程中，代码通常按照一定的顺序、在一定的时机进行调用，也就是控制权通常由开发者手动控制。而 IOC 模式则是将控制权交给了框架或容器，由框架或容器自动执行模块的方法，模块对外部环境的依赖通过外部注入而不是自己创建，这样使模块之间的依赖更加灵活和可维护。IOC 常见的实现方式包括依赖注入（DI）和控制反转容器（IOC 容器）。

### Nest 中的 IOC

#### Service

Service 类上声明了`@Injectable`，表明这个类是可注入的，那么 Nest 就会把它放到 IOC 容器中。

```typescript
@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

#### Controller

Controller 类上声明了`@Controller`，表明这个类也是可注入的，Nest 也会把它放到 IOC 容器中。

```typescript
import { AppService } from './AppService';

@Controller()
export class AppController() {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

为什么 Controller 是单独的装饰器？

因为 Service 是可以被注入，也可以注入到别的对象的，所以用`@Injectable`声明。而 Controller 只需要被注入，所以 Nest 单独给它提供了`@Controller`装饰器。

#### Module

Module 类上声明了`@Module`，表面这个类是一个模块。Nest 使用模块来组织应用程序结构。

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

Nest 会从 AppModule（根模块）开始解析类上通过装饰器声明的依赖信息，自动创建和组装对象。所以 Controller 类中只是声明了对 Service 类的依赖，就能获取到它。

Nest 的模块机制，可以把不同业务的 Controller、Service 等放到不同模块里。当引入别的的模块后，引入模块导出的 provider 就可以在当前模块进行注入。

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

示例的中 AppModule 内部可以使用 OtherModule 导出的 OtherService 来进行注入。
