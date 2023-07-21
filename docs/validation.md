## 参数验证和转换

Pipe 有两个典型的用例：

- 验证：评估输入数据，如果有效，就原封不动的专递它；否则就抛出异常。

- 转换：能将输入的数据转换为所需的形式（例如，从字符串转为整数）。

### GET 请求参数

#### 路径参数

GET 请求的路径参数通常不需要校验，且值默认都是字符串。

```typescript
@Get(':id')
findOne(@Param('id') id: number) {
  console.log(typeof id); // => string
}

// testing: '/123'
```

使用 Pipe 对路径参数进行转换，若不能成功转换类型，则抛出异常。

```typescript
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  console.log(typeof id); // => number
}

// testing: '/123'
```

#### 查询字符串

查询字符串也能通过使用 Pipe 对参数进行转换。

```typescript
@Get()
findAll(@Query('limit', ParseIntPipe) limit: number) {
  console.log(typeof limit) // => number
}
```

查询字符串通常还会以对象的形式进行验证和转换，这需要安装两个**生产依赖包**：

- class-validator: 提供基于装饰器声明规则的对象验证功能。

```typescript
import { IsString, IsNumber } from 'class-validator';

export class PaginationQueryDto {
  @IsOptional()
  @IsPositive()
  limit: number;

  @IsOptional()
  @IsPositive()
  offset: number;
}
```

- class-transformer: 将普通对象转换为某个类的实例对象。

默认情况下 DTO 只为参数提供类型，并不会对参数进行任何处理。因此需要搭配 `ValidationPipe` 来进行校验和转换。

```typescript
@Get()
@UsePipes(
  new ValidationPipe({
    transform: true, // 转换为 DTO 类的实例
    transformOptions: {
      enableImplicitConversion: true // 启用隐式数据类型转换，确保验证通过的数据与 DTO 类的属性类型匹配
    }
  })
)
findAll(@Query() paginationQuery: PaginationQueryDto) {
  console.log(paginationQueryDto instanceof PaginationQueryDto) // => true
}

// testing data /?limit=5&offset=1
```

### POST 请求参数

POST 请求的参数也可以使用 DTO 和 `ValidationPipe` 来验证和转换。

```typescript
// create-cat.dto.ts
export class CreateCatDto {
  name: string;
  age: number;
}

@Post()
@UsePipes(new ValidationPipe({
  whitelist: true, // 去除未在 DTO 类中使用验证装饰器定义的属性
  transform: true,
  transformOptions: {
    enableImplicitConversion: true
  }
}))
create(@Body() createCatDto: CreateCatDto) {
  console.log(typeof createCatDto.age); // => number
  console.log(createCatDto instanceof CreateCatDto); // false
}

testing data: { name: 'Tom', age: '4' }
```
