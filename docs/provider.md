## Provider

### useClass

provider 需要在 Module 的 providers 里声明。使用类创建的 provider 通过 provide 指定注入的 token，通过 useClass 指定注入的类。

```typescript
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

其实这是一种简写，完整的写法是这样的：

```typescript
@Module({
  imports: [],
  controllers: [AppController],
  providers: [
    {
      provide: AppService,
      useClass: AppService,
    },
  ],
})
export class AppModule {}
```

Nest 会自动对 provider 做实例化再注入到可注入类中。

```typescript
@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}
}
```

如果不想用构造器注入，也可以用属性注入，通过 `@Inject` 指定注入的 provider 的 token 即可。

```typescript
@Controller()
export class AppController() {
  @Inject(AppService)
  private readonly appService: AppService;
}
```

provider 的 token 也可以是字符串：

```typescript
@Module({
  imports: [],
  controllers: [AppController],
  providers: [
    {
      provider: 'AppService',
      useClass: AppService,
    },
  ],
})
export class AppModule {}

@Controller()
export class AppController {
  constructor(@Inject('AppService') private readonly appService: AppService) {}
}
```

如果 token 是字符串，注入时必须使用 `@Inject` 手动指定注入对象的 token。相比之下，使用类做 token 可以省去 `@Inject` ，比较简便，所以更推荐使用。

### useValue

除了指定类外，还可以直接指定一个值作为 provider 让 IOC 容器来注入。通过 provider 指定 token，通过 useValue 指定值。

```typescript
@Module({
  imports: [],
  controllers: [AppController],
  providers: [
    {
      provider: 'user',
      useValue: {
        name: 'jee',
        age: 24,
      },
    },
  ],
})
export class AppModule {}

@Controller()
export class AppController {
  constructor(@Inject('user') private readonly user: { name: string; age: number }) {}
}
```

### useFactory

provider 的值有时可能是动态产生的，Nest 同样也能支持。

```typescript
@Module({
  imports: [],
  controllers: [AppController],
  providers: [
    {
      provider: 'user',
      useFactory() {
        return {
          name: 'jee',
          age: 24,
        };
      },
    },
  ],
})
export class AppModule {}

@Controller()
export class AppController {
  constructor(@Inject('user') private readonly user: { name: string; age: number }) {}
}
```

useFactory 支持参数注入，能将声明在 providers 中的其他依赖注入到参数中。

```typescript
@Module({
  imports: [],
  controllers: [AppController],
  providers: [
    AppService,
    {
      provider: 'user',
      useValue: {
        name: 'jee',
        age: 24,
      },
    },
    {
      provider: 'person',
      useFactory(user: { name: string }, appService: AppService) {
        return {
          name: user.name,
          desc: appService.getHello(),
        };
      },
    },
  ],
})
export class AppModule {}
```

useFactory 还支持异步，Nest 会等拿到异步方法的结果之后再注入。

```typescript
@Module({
  imports: [],
  controllers: [AppController],
  providers: [
    {
      provider: 'user',
      async useFactory() {
        await new Promise((resolve) => {
          setTimeout(resolve, 3000);
        });
        return {
          name: 'jee',
          age: 24,
        };
      },
    },
  ],
})
export class AppModule {}
```

### useExisting

useExisting 用于给 provider 指定别名：

```typescript
@Module({
  imports: [],
  controllers: [AppController],
  providers: [
    {
      provider: 'user',
      useValue: {
        name: 'jee',
        age: 24,
      },
    },
    {
      provider: 'user',
      useExiting: 'person',
    },
  ],
})
export class AppModule {}

@Controller()
export class AppController {
  constructor(@Inject('person') private readonly person: { name: string; age: number }) {}
}
```

示例中就是给 user 起了一个别名 person，然后就可以用这个新的 token 去注入了。
