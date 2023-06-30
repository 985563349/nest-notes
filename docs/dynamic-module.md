## 动态模块

当需要在使用模块时传递一些参数，动态生成模块内容时，就可以使用动态模块。

### 定义动态模块

定义动态模块，需要实现一个注册模块的静态方法，按照惯例一般会将该方法命名为：`register` 或者 `forRoot`。

注册方法的返回内容必须符合 `DynamicModule` 接口。动态模块只是在运行时创建的，具有和静态模块（通过 `@Module` 装饰器注册的模块）完全相同的属性，外加一个名为 `module` 的附加属性。

```typescript
@Module({})
export class ConfigModule {
  static register(options: Record<string, any>): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        ConfigService,
        {
          provider: 'CONFIG_OPTIONS',
          useValue: options,
        },
      ],
      exports: [ConfigService],
    };
  }
}
```

### 注册动态模块

`imports` 中调用动态模块的注册方法，就可以完成模块的注册，同时还能传递一些参数给模块。

```typescript
@Module({
  imports: [ConfigModule.register({ folder: './config' })],
})
export class AppModule {}
```

### 注册方法名

关于动态模块的注册方法名并没有严格规定，但 Nest 推荐使用以下三种方法名，以在不同场景下使用：

- register：每次配置注册都产生新的模块。
- forRoot：配置一次，在多个地方重用（通常是在根模块中注册）。
- forFeature：使用 `forRoot` 的公共配置，在局部使用时额外再传入一些配置（比如用 `forRoot` 指定数据库链接信息，再用 `forFeature` 指定模块访问哪些数据库和表）。

需要异步的场景下可以命名为：`registerAsync`、`forRootAsync` 和 `forFeatureAsync`。

### 动态模块构建器

Nest 提供了一个动态模块构建器：`ConfigurableModuleBuilder`，用于快速生成动态模块。

```typescript
// config-module.options.interface.ts
export interface ConfigModuleOptions {
  folder: string;
}

// config.module.definition.ts
export const { ConfigurableModuleClass, MODULE_OPTIONS_TOKEN } =
  new ConfigurableModuleBuilder<ConfigModuleOptions>().build();

// config.service.ts
@Injectable()
export class ConfigService {
  constructor(@Inject(MODULE_OPTIONS_TOKEN) private readonly options: ConfigModuleOptions) {}
}

// config.module.ts
@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule extends ConfigurableModuleClass {}
```

`ConfigurableModuleBuilder` 生成一个类，这个类默认提供了 `register` 和 `registerAsync` 两个静态方法。还有一个模块配置参数的 provider token。

`registerAsync` 方法可以使用 `useFactory` 动态创建模块配置对象。

```typescript
@Module({
  imports: [
    ConfigModule.registerAsync({
      useFactory: () => {
        return {
          folder: './config',
        };
      },
    }),
  ],
})
export class AppModule {}
```

#### 修改注册方法名

调用 `setClassMethodName` 可以修改默认的注册方法名。

```typescript
new ConfigurableModuleBuilder<ConfigModuleOptions>().setClassMethodName('forRoot').build();

@Module({
  imports: [ConfigModule.forRoot({ folder: './config' })],
})
export class AppModule {}
```

#### 额外配置参数

调用 `setExtras` 可以给模块注册配置添加额外的选项。例如，在模块配置中加入 isGlobal 选项决定该模块是否设置为全局模块。

```typescript
new ConfigurableModuleBuilder<ConfigModuleOptions>()
  .setExtras({ isGlobal: true }, (definition, extras) => ({
    ...definition,
    global: extras.isGlobal,
  }))
  .build();

@Module({
  imports: [ConfigModule.register({ folder: './config', isGlobal: true })],
})
export class AppModule {}
```

注入时使用的 `ConfigModuleOptions` 定义类型在默认情况下无法注入额外的配置。如果需要使用到额外的配置项，可以使用 `OPTIONS_TYPE` 或 `ASYNC_OPTIONS_TYPE` 类型来注入。

```typescript
const { ConfigurableModuleClass, MODULE_OPTIONS_TOKEN, OPTIONS_TYPE, ASYNC_OPTIONS_TYPE } =
  new ConfigurableModuleBuilder<ConfigModuleOptions>().build();

@Injectable()
export class ConfigService {
  constructor(@Inject(MODULE_OPTIONS_TOKEN) private readonly options: typeof OPTIONS_TYPE) {}
}
```
