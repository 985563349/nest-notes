## 循环依赖

当两个类相互依赖时，就会发生循环依赖。在 Nest 中，模块之间和提供者之间可能会出现循环依赖的问题。

实际开发中应尽可能的避免循环依赖，在不可避免的情况下，Nest 提供了转发引用的方式来解决循环依赖问题。

### 转发引用

例如，`CatService` 和 `CommonService` 互相依赖，关系的双方可以使用 `@Inject` 和 `forwardRef` 来解决循环依赖。

```typescript
@Injectable()
export class CatService {
  constructor(
    @Inject(forwardRef(() => CommonService))
    private readonly commonService: CommonService
  ) {}
}

@Injectable()
export class CommonService {
  constructor(
    @Inject(forwardRef(() => CatService))
    private readonly catService: CatService
  ) {}
}
```

### 模块引用转发

模块之前的循环依赖，可以在模块关联的两侧转发引用。

```typescript
@Module({
  imports: [forwardRef(() => CatsModule)],
})
export class CommonModule {}

@Module({
  imports: [forwardRef(() => CommonModule)],
})
export class CatsModule {}
```
