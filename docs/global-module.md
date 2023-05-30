## 全局模块

模块导出 provider，另一个模块需要导入它才能使用。如果这个模块被很多模块依赖，就可以将它定义为全局模块。

全局模块通过 `@Global` 修饰符定义，被定义模块导出的 provider 能直接被注入，而不需要导入这个模块。

```typescript
@Global()
@Module({
  imports: [],
  controllers: [GlobalController],
  providers: [GlobalService],
  exports: [GlobalService],
})
export class GlobalModule {}

@Controller()
export class AppController {
  constructor(private readonly globalService, private readonly appService) {}
}
```

全局模块应尽量减少使用，过多的全局模块会导致注入的 provider 会无法追踪来源，导致代码可维护性降低。
