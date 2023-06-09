## 数据传输

### 数据格式

前后端数据传输的格式主要有五种：

- URL Param
- Query
- Form URLEncoded
- Form Data
- JSON

#### URL Param

将参数写在 url 中，例如：

```ini
http://localhost:8080/api/person/1
```

url 中的 1 就是路径参数（url param），服务端框架和单页应用的路由系统都支持从 url 中读取参数。

#### Query

通过 url 中 **?** 后面的用 **&** 分隔的字符串传递数据，例如：

```ini
http://localhost:8080/api/person?name=jee&age=24
```

url 中的 name 和 age 就是 query 传递的参数，其中非英文的字符和一些特殊字符需要进行编码。

可以使用 `encodeURIComponent` 方法来编码：

```javascript
const query = '?name=' + encodeURIComponent('王花花') + '&age=' + encodeURIComponent(24);
// => '?name=%E7%8E%8B%E8%8A%B1%E8%8A%B1&age=24'
```

也可以使用第三方库 `query-string` 来处理：

```javascript
const queryString = require('query-string');
const query = queryString.stringify({ name: '王花花', age: 24 });
// => '?name=%E7%8E%8B%E8%8A%B1%E8%8A%B1&age=24'
```

#### Form URLEncoded

前端使用 form 表单提交的数据就是这种格式，它与 query 字符串类似，区别在于 form urlencoded 是放在请求体里，并且指定了请求头参数 **content-type: application/x-www-form-urlencoded**。

由于内容也是 query 字符串，所以对特殊字符也需要进行编码。

#### Form Data

form data 通常用于做文件传输。 请求头参数 **content-type** 会定义数据格式为： **multipart/form-data**，以及数据的分隔符 （boundary，由短横线加数字组成的字符串）。

#### JSON

form urlencoded 会对内容进行编码，form data 会增加 boundary 导致请求体积增大，如果只是传输简单数据，推荐使用 json。

传输 json 数据时，需要指定请求头参数 **content-type: application/json**。

---

### Nest 接收数据

#### URL Param

url 中的参数，Nest 里通过`@Param`装饰器来注入。

```typescript
@Controller('api/person')
export class PersonController {
  @Get(':id')
  findOne(@Param('id') id: string) {
    return `received: id =${id}`;
  }
}
```

`@Controller('api/person')`定义的路由和`@Get(':id')`定义的路由会拼接在一起，即只有路径为 `/api/person/xxx` 的 GET 请求才会调用这个函数。

#### Query

url 中的 query 字符串会做编码，在 Nest 里通过`@Query`装饰器来注入。

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

form urlencoded 将数据放在了请求体中，并且做了编码，Nest 接受需要使用`@Body`装饰器。同时 Nest 还会解析请求体，并注入到 dto 中。

dto（data transfer object） 用于封装传输数据的对象。

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

Nest 解析 form data 需要用到 `AnyFilesInterceptor`拦截器，再通过`@UseInterceptors`装饰器来启用拦截器。文件内容通过`@UploadedFiles`装饰器注入，非文件内容通过`@Body`装饰器注入。

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

> TIP: 类型`Express.Multer.File`依赖于第三方包`@types/multer`。

#### JSON

Nest 接收 form urlencoded 和 json 都使用`@Body`装饰器，Nest 的内部会根据请求头参数 **content-type** 来选择不同的解析方式。

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
