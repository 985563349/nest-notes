## MySQL 基础命令

在书写 MySQL 命令时需要注意以下几点：

- 大小写没有限制。
- 字符串可以使用单引号，或者双引号包裹。
- 必须使用结束符号结尾。

MySQL 命令结束符号通常为分号 ，也可以使用 **\g**、 **\G** 和 **\c** 结尾，其中 **\c** 结尾表示放弃本次编写的命令。

### 库操作

#### 展示数据库

`SHOW DATABASES` 命令可以查询当前 MySQL 服务中的所有数据库。

```mysql
SHOW DATABASES;
```

#### 查看当前数据库

`SELECT DATABASE` 命令可以查询当前正在使用的数据库。

```mysql
SELECT DATABASE;
```

#### 切换数据库

`USE` 命令可以切换当前使用的数据库。

```mysql
USE <database-name>;
```

#### 创建数据库

`CREATE DATABASE` 命令可以创建新的数据库。

```mysql
CREATE DATABASE <database-name>;
```

#### 修改数据库名

`RENAME DATABASE` 可以修改数据库名。

```mysql
RENAME DATABASE <old-database-name> TO <old-database-name>;
```

#### 删除数据库

`DROP DATABASE` 命令可以删除数据库。

```mysql
DROP DATABASE <database-name>;
```

### 表操作

### 展示数据表

`SHOW TABLES` 命令可以查看当前数据库中的所有表。

```mysql
SHOW TABLES;
```

#### 查看表结构

`DESC` 命令可以查看表结构。

```mysql
DESC <table-name>;
```

#### 查看建表语句

`SHOW CREATE TABLE` 命令可以查看建表语句。

```mysql
SHOW CREATE TABLE <table-name>;
```

#### 修改表名

修改表名有两种方式：

方式一：`ALTER TABLE` 命令。

```mysql
ALTER TABLE <old-table-name> TO <new-table-name>
```

方式二：`RENAME TABLE` 命令。

```mysql
RENAME TABLE <old-table-name> TO <new-table-name>;
```

#### 删除表

`DROP TABLE` 命令可以删除表。

```mysql
DROP TABLE <table-name>;
```
