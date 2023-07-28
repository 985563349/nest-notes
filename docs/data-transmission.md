## 数据传输

### 数据格式

前后端数据传输的格式主要有五种：

- 路径参数
- Query String
- Form URLEncoded
- Form Data
- JSON

#### 路径参数

将参数写在 URL 中，例如：

```ini
http://localhost:8080/api/person/1
```

URL 中的数字 1 就是路径参数，服务端框架和单页应用的路由系统都支持从 URL 中读取参数。

#### Query String

通过使用问号 (?) 将参数附加到 URL 末尾，并使用(&) 符号分隔不同的参数，例如：

```ini
http://localhost:8080/api/person?name=jee&age=24
```

URL 中的 name=jee 和 age=24 就是传递的参数。参数值中非英文的字符和一些特殊字符需要进行编码。

使用 `encodeURIComponent` 方法进行编码：

```javascript
const query = '?name=' + encodeURIComponent('王花花') + '&age=' + encodeURIComponent(24);
// => '?name=%E7%8E%8B%E8%8A%B1%E8%8A%B1&age=24'
```

也可以使用第三方库 `query-string` ：

```javascript
const queryString = require('query-string');
const query = queryString.stringify({ name: '王花花', age: 24 });
// => '?name=%E7%8E%8B%E8%8A%B1%E8%8A%B1&age=24'
```

#### Form URLEncoded

前端使用 Form 表单提交的数据就是这种格式，它与 Query String 类似（数据格式与编码方式相同），区别在于 Form URLEncoded 的数据是放在请求体里，并且需要指定请求头参数 `Content-Type` 为 `application/x-www-form-urlencoded`。

#### Form Data

Form Data 通常用来做文件传输。 请求头参数 **Content-Type** 会定义数据格式为： **multipart/form-data**，以及数据的分隔符 （boundary，由短横线加数字组成的字符串）。

#### JSON

Form URLEncoded 会对内容进行编码，Form Data 会增加 boundary 导致请求体积增大，如果只是传输简单数据，推荐使用 JSON。

传输 JSON 数据时，需要指定请求头参数 **Content-Type: application/json**。

### Nest 接收数据

#### 路径参数

路径中的参数，Nest 可以通过 `@Param` 来接收。

```typescript
@Controller('api/person')
export class PersonController {
  @Get(':id')
  findOne(@Param('id') id: string) {
    return `received: id = ${id}`;
  }
}
```

`@Controller('api/person')` 定义的路由和 `@Get(':id')` 定义的路由会拼接在一起，即只有路径为 `/api/person/xxx` 的 GET 请求才会调用这个函数。

#### Query String

Query String 会对数据内容做编码，Nest 里通过 `@Query` 来接收，装饰器会自动进行解码。

```typescript
@Controller('api/person')
export class PersonController {
  @Get('find')
  find(@Query('name') name: string, @Query('age') age: number) {
    return `received: name=${name},age=${age}`;
  }
}
```

#### Form URLEncoded

Form URLEncoded 将数据放在了请求体中，Nest 里通过 `@Body` 来接收。同时 Nest 还会解析请求体，并注入到 DTO 中。

DTO（data transfer object） 用于封装传输数据的对象。

```typescript
// create-person.dto.ts
export class CreatePersonDto {
  name: string;
  age: number;
}

// person.controller.ts
import { CreatePersonDto } from './dto/create-person.dto';

@Controller('api/person')
export class PersonController {
  @Post()
  create(@Body() createPersonDto: CreatePersonDto) {
    return `received: ${JSON.stringify(createPersonDto)}`;
  }
}
```

#### Form Data

Nest 解析 Form Data 需要使用 `AnyFilesInterceptor` 拦截器，再通过 `@UseInterceptors` 装饰器来启用拦截器。

文件内容通过 `@UploadedFiles` 来接收，非文件内容通过 `@Body` 来接收。

```typescript
import { AnyFilesInterceptor } from '@nestjs/platform-express';
import { CreatePersonDto } from './dto/create-person.dot';

@Controller('api/person')
export class PersonController () {
  @Post('file')
  @UseInterceptor(AnyFilesInterceptor())
  upload(@Body() createPersonDto: CreatePersonDto, @UploadedFiles() files: Array<Express.Multer.File>) {
    console.log(files);
    return `received: ${JSON.stringify(createPersonDto)}`;
  }
}
```

类型 `Express.Multer.File` 依赖于第三方包 `@types/multer` 。

#### JSON

Nest 接收 Form URLEncoded 和 JSON 都使用 `@Body`，Nest 的内部会根据请求头参数 **Content-Type** 来选择不同的解析方式。

```typescript
import { CreatePersonDto } from './dto/create-person.dot';

@Controller('api/person')
export class PersonController () {
  @Post()
  create(@Body() createPersonDto: CreatePersonDto) {
    return `received: ${JSON.stringify(createPersonDto)}`;
  }
}
```
