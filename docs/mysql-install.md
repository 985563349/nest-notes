## MySQL 安装

使用 Docker 可以很方便的安装 MySQL。

### 查询镜像

使用 `docker search mysql` 命令查询 MySQL 当前可用的镜像：

```shell
docker search mysql
```

### 拉取镜像

使用 `docker pull mysql` 命令拉取 MySQL 最新版本的镜像：

```shell
docker pull mysql
```

### 运行容器

镜像拉取完成后，可以使用 `docker run` 命令来运行 MySQL 容器：

```shell
docker run -itd --name hello-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /var/lib/mysql:/var/lib/mysql mysql
```

参数说明：

- **-p**：映射容器的端口到宿主机端口。
- **-e**：指定容器服务的环境变量，用于设置 MySQL 服务 root 用户的密码。
- **-v**：将本地目录作为数据卷挂载到容器。

### 连接 MySQL

通过 `docker exec` 命令进入 MySQL 容器：

```shell
docker exec -it <container-id> bash
```

使用 root 用户连接到 MySQL 服务：

```shell
mysql -u root -p
```

输入密码后，就可以成功连接到 MySQL 服务。
