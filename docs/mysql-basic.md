## MySQL 基础

MySQL 是当下最流行的关系型数据库。

### 安装

使用 Docker 可以很方便的安装 MySQL。

#### 查询镜像

使用 `docker search mysql` 命令查询 MySQL 当前可用的镜像：

```shell
docker search mysql
```

#### 拉取镜像

使用 `docker pull mysql` 命令拉取 MySQL 最新版本的镜像：

```shell
docker pull mysql
```

#### 运行容器

镜像拉取完成后，可以使用 `docker run` 命令来运行 MySQL 容器：

```shell
docker run -itd --name hello-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /var/lib/mysql:/var/lib/mysql mysql
```

参数说明：

- **-p**：映射容器的端口到宿主机端口。
- **-e**：指定容器服务的环境变量，用于设置 MySQL 服务 root 用户的密码。
- **-v**：将本地目录作为数据卷挂载到容器。

#### 连接 MySQL

通过 `docker exec` 命令进入 MySQL 容器：

```shell
docker exec -it <container-id> bash
```

使用 root 用户连接到 MySQL 服务：

```shell
mysql -u root -p
```

输入密码后，就可以成功连接到 MySQL 服务。

### 基础命令

在书写 MySQL 命令时需要注意以下几点：

- 大小写没有限制。
- 字符串可以使用单引号，或者双引号包裹。
- 必须使用结束符号结尾。

MySQL 命令结束符号通常以分号 ，也可以使用 **\g** 或者 **\G** 结尾。

#### 展示数据库

使用 `show databases;` 命令可以查询当前 MySQL 服务中的所有数据库。

```mysql
show databases;
```

#### 查看当前数据库

使用 `select database;` 命令可以查询当前正在使用的数据库。

```mysql
select database;
```

#### 切换数据库

使用 `use` 命令可以切换当前使用的数据库。

```mysql
use <database-name>;
```

#### 创建数据库

使用 `create database` 命令可以创建新的数据库。

```mysql
create database <database-name>;
```

#### 删除数据库

使用 `drop database` 命令可以删除数据库。

```mysql
drop database <database-name>;
```
