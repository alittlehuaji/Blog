`docker container exec` 的核心作用是在一个正在运行的容器中执行命令

使用方法：

```bash
docker container exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

还有另外一种写法是

```bash
docker exec ...
```

本文将围绕`docker exec`这种写法来进行讲解

最常用形式：

```bash
docker exec -it CONTAINER bash
docker exec -it CONTAINER sh
```

常用参数记忆：

```text
-i              保持输入打开，适合交互或重定向输入
-t              分配伪终端，适合进入 Shell
-it             最常见，交互式进入容器
-d              后台执行命令
-e              临时设置环境变量
--env-file      从文件读取环境变量
-u              指定执行用户
-w              指定工作目录
--privileged    给本次 exec 命令更高权限
```

最重要的几个注意点：

- `docker exec` 只能用于运行中的容器
- 进入容器常用 `docker exec -it CONTAINER sh`
- 复杂命令要用 `sh -c '...'`
- 脚本、管道、重定向场景通常不要加 `-t`
- `docker exec` 中的路径是容器内部路径

## 基本作用

`docker exec` 用于在一个**已经运行中的容器**里执行命令

基本语法：

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

例如：

```bash
docker exec my-nginx nginx -v
```

含义是：在名为 `my-nginx` 的运行中容器里执行 `nginx -v`

等价写法：

```bash
docker container exec my-nginx nginx -v
```

## 前提条件

`docker exec` 只能作用于**正在运行的容器**。

先查看运行中的容器：

```bash
docker ps
```

示例输出：

```text
CONTAINER ID   IMAGE     COMMAND                  NAMES
abc123456789   nginx     "/docker-entrypoint.…"   my-nginx
```

然后可以使用容器名或容器 ID 执行命令：

```bash
docker exec my-nginx ls
docker exec abc123456789 ls
```

如果容器已经停止，`docker exec` 会失败

```bash
docker exec my-nginx ls
```

可能报错：

```text
Error response from daemon: Container ... is not running
```

停止状态下应使用：

```bash
docker start my-nginx
```

然后再执行：

```bash
docker exec my-nginx ls
```

## 最常用：进入容器 Shell

### 进入 Bash

```bash
docker exec -it CONTAINER bash
```

例如：

```bash
docker exec -it my-app bash
```

其中：

- `-i`：保持标准输入打开
- `-t`：分配一个伪终端
- `bash`：进入容器中的 Bash Shell

这是最常见的组合

### 进入 sh

有些精简镜像没有 `bash`，例如 Alpine、BusyBox以及部分 distroless 镜像

可以使用：

```bash
docker exec -it CONTAINER sh
```

例如：

```bash
docker exec -it alpine-container sh
```

## 执行单条命令

如果只需要执行单条命令，不需要进入 Shell，可以直接执行命令

```bash
docker exec CONTAINER COMMAND
```

例如查看目录：

```bash
docker exec my-nginx ls /usr/share/nginx/html
```

## `-i` 和 `-t` 的区别

### `-i`：保持 STDIN 打开

```bash
docker exec -i CONTAINER COMMAND
```

常用于需要从标准输入传递内容的场景。

例如导入 SQL：

```bash
docker exec -i mysql-container mysql -uroot -p123456 mydb < backup.sql
```

这里必须用 `-i`，否则标准输入可能无法传入容器。

### `-t`：分配伪终端

```bash
docker exec -t CONTAINER COMMAND
```

用于需要终端交互体验的命令。

### `-it`：交互式终端

最常用组合：

```bash
docker exec -it CONTAINER bash
```

适合进入容器进行交互式操作。

### 什么时候不要用 `-t`

如果使用管道、重定向、脚本自动化，通常不要加 `-t`

例如：

```bash
docker exec -i mysql-container mysql -uroot -p123456 mydb < backup.sql
```

不要写成：

```bash
docker exec -it mysql-container mysql -uroot -p123456 mydb < backup.sql
```

否则可能出现：

```text
the input device is not a TTY
```

总结

- 交互式登录容器：用 `-it`

- 管道/重定向输入：用 `-i`

- 普通执行命令：**通常不用** `-i -t`

## 后台执行命令：`-d`

`-d` 表示 detached mode，即后台执行

```bash
docker exec -d CONTAINER COMMAND
```

例如在容器中后台执行脚本：

```bash
docker exec -d my-app sh -c "python /app/task.py"
```

或者：

```bash
docker exec -d my-app sh -c "sleep 60 && echo done > /tmp/done.log"
```

注意：

- `docker exec -d` **不会把命令输出显示在当前终端**
- 如果需要查看结果，应将输出写入文件或通过容器日志查看

## 指定环境变量：`-e`

可以给本次执行的命令临时设置环境变量。

```bash
docker exec -e KEY=VALUE CONTAINER COMMAND
```

例如：

```bash
docker exec -e APP_ENV=debug my-app env
```

多个环境变量：

```bash
docker exec \
  -e APP_ENV=debug \
  -e LOG_LEVEL=trace \
  my-app env
```

在 Shell 中使用：

```bash
docker exec -e NAME=huajibenji my-app sh -c 'echo $NAME'
```

注意这里建议使用单引号：

```bash
sh -c 'echo $NAME'
```

如果写成双引号：

```bash
sh -c "echo $NAME"
```

`$NAME` 可能会先被宿主机 Shell 展开，而不是在容器内展开。

## 使用环境变量文件：`--env-file`

可以从文件读取环境变量。

```bash
docker exec --env-file ./exec.env CONTAINER COMMAND
```

`exec.env` 示例：

```env
APP_ENV=debug
LOG_LEVEL=debug
USERNAME=huajibenji
```

执行：

```bash
docker exec --env-file ./exec.env my-app env
```

注意：

- `--env-file` 中的变量**只对本次** `docker exec` 命令生效
- **不会**永久修改容器环境变量
- **不会**修改镜像或 Dockerfile

## 指定用户执行：`-u`

默认情况下，`docker exec` 使用容器的默认用户执行命令。

可以用 `-u` 指定用户：

```bash
docker exec -u USER CONTAINER COMMAND
```

例如使用 root：

```bash
docker exec -u root -it my-app bash
```

使用普通用户：

```bash
docker exec -u appuser -it my-app sh
```

也可以使用 UID：

```bash
docker exec -u 1000 my-app whoami
```

指定用户和组：

```bash
docker exec -u 1000:1000 my-app id
```

格式：

```text
<name|uid>[:<group|gid>]
```

常见用途：

```bash
docker exec -u root my-app apt update
docker exec -u www-data my-app php artisan cache:clear
docker exec -u postgres postgres-container psql
```

注意：

- 指定的用户必须在容器内存在，或者使用 UID/GID
- 使用 root 执行命令要谨慎，可能会改变文件权限

## 指定工作目录：`-w`

`-w` 或 `--workdir` 用于指定命令在容器内的工作目录。

```bash
docker exec -w /path/in/container CONTAINER COMMAND
```

例如：

```bash
docker exec -w /app my-app pwd
```

输出：

```text
/app
```

执行项目命令：

```bash
docker exec -w /app my-app npm run build
```

进入指定目录的 Shell：

```bash
docker exec -it -w /app my-app bash
```

等价于进入容器后执行：

```bash
cd /app
```

## 使用 Shell 执行复杂命令

如果要执行复杂命令，比如管道、重定向、变量展开、条件判断，则需要通过 Shell

错误示例：

```bash
docker exec my-app echo hello > /tmp/a.txt
```

这个 `>` 是在宿主机执行的，不是在容器内执行

正确写法：

```bash
docker exec my-app sh -c 'echo hello > /tmp/a.txt'
```

### 执行多条命令

```bash
docker exec my-app sh -c 'cd /app && npm install && npm run build'
```

或者：

```bash
docker exec my-app sh -c '
cd /app
npm install
npm run build
'
```

### 使用管道

```bash
docker exec my-app sh -c 'ps aux | grep nginx'
```

### 使用重定向

```bash
docker exec my-app sh -c 'echo "hello" > /tmp/hello.txt'
```

追加内容：

```bash
docker exec my-app sh -c 'echo "world" >> /tmp/hello.txt'
```

### 条件执行

```bash
docker exec my-app sh -c 'test -f /app/config.yml && echo exists || echo missing'
```

## 使用特权模式：`--privileged`

`--privileged` 会给本次 `exec` 执行的命令更高权限

```bash
docker exec --privileged CONTAINER COMMAND
```

例如：

```bash
docker exec --privileged my-container iptables -L
```

或：

```bash
docker exec --privileged -it my-container sh
```

注意：

- `--privileged` 只影响这次 `exec` 执行的进程
- 不等于整个容器以 privileged 模式运行
- 安全风险较高，应谨慎使用
- 容器本身如果没有相关能力或挂载，某些操作仍可能失败

通常只有在排查网络、内核能力、设备访问等问题时才考虑使用

## 修改 detach 快捷键：`--detach-keys`

当使用交互式终端时，默认可以通过快捷键脱离容器终端

常见默认快捷键是：

```text
Ctrl-p Ctrl-q
```

可以用 `--detach-keys` 自定义：

```bash
docker exec -it --detach-keys="ctrl-x" my-app bash
```

这样可以用 `Ctrl-x` 脱离当前 exec 会话

注意：

- 脱离不是停止容器
- 该操作只是退出当前终端连接

## 常见实际场景

### 进入正在运行的容器

```bash
docker exec -it my-app bash
```

如果没有 Bash：

```bash
docker exec -it my-app sh
```

### 查看容器内文件

```bash
docker exec my-app ls -lah /app
```

```bash
docker exec my-app cat /app/config.yml
```

### 查看容器内进程

```bash
docker exec my-app ps aux
```

### 查看端口监听

```bash
docker exec my-app netstat -tunlp
```

或者：

```bash
docker exec my-app ss -tunlp
```

有些精简镜像可能没有这些命令

### 测试容器内网络

```bash
docker exec my-app ping bing.com
```

```bash
docker exec my-app curl http://localhost:8080
```

```bash
docker exec my-app wget -qO- http://localhost:8080
```

注意：

- `localhost` 指的是容器内部自身
- 不是宿主机
- 不是其他容器

### 在容器内执行项目命令

Node.js：

```bash
docker exec -w /app my-node npm install
docker exec -w /app my-node npm run build
```

Python：

```bash
docker exec -w /app my-python python manage.py migrate
```

PHP Laravel：

```bash
docker exec -w /var/www/html my-php php artisan migrate
```

Java：

```bash
docker exec my-java java -version
```

### 执行数据库命令

MySQL：

```bash
docker exec -it mysql-container mysql -uroot -p
```

直接执行 SQL：

```bash
docker exec mysql-container mysql -uroot -p123456 -e "SHOW DATABASES;"
```

导入 SQL：

```bash
docker exec -i mysql-container mysql -uroot -p123456 mydb < backup.sql
```

导出 SQL：

```bash
docker exec mysql-container mysqldump -uroot -p123456 mydb > backup.sql
```

PostgreSQL：

```bash
docker exec -it postgres-container psql -U postgres
```

执行 SQL：

```bash
docker exec postgres-container psql -U postgres -c '\l'
```

Redis：

```bash
docker exec -it redis-container redis-cli
```

执行 Redis 命令：

```bash
docker exec redis-container redis-cli ping
```

### 查看容器内环境变量

```bash
docker exec my-app env
```

查看某个变量：

```bash
docker exec my-app sh -c 'echo $PATH'
```

### 在容器中创建文件

```bash
docker exec my-app sh -c 'echo hello > /tmp/hello.txt'
```

追加内容：

```bash
docker exec my-app sh -c 'echo world >> /tmp/hello.txt'
```

查看：

```bash
docker exec my-app cat /tmp/hello.txt
```

## `docker exec` 与 `docker run` 的区别

### docker exec

在已有运行中容器里执行命令：

```bash
docker exec -it my-app bash
```

特点：

- 容器必须已经运行
- 不会创建新容器
- 适合调试、排查、临时执行命令

### docker run

基于镜像创建并启动新容器：

```bash
docker run -it ubuntu bash
```

特点：

- 会创建新容器
- 可以从镜像启动
- 适合启动服务或创建临时容器

对比：

```text
docker exec：进入或操作一个已经存在且运行中的容器
docker run ：基于镜像创建一个新的容器
```

## `docker exec` 与 `docker attach` 的区别

### docker exec

启动一个新的进程：

```bash
docker exec -it my-app bash
```

特点：

- 在容器内新开一个命令进程
- 常用于进入容器调试
- 退出 Shell 不会影响容器主进程

### docker attach

连接到容器的主进程：

```bash
docker attach my-app
```

特点：

- 附加到容器当前主进程的标准输入输出
- 如果退出方式不当，可能影响容器主进程
- 不如 `docker exec` 常用于调试

日常进入容器优先使用：

```bash
docker exec -it CONTAINER bash
```

而不是：

```bash
docker attach CONTAINER
```

## 常见错误与解决方法

### 容器没有运行

错误：

```text
Error response from daemon: Container ... is not running
```

解决：

```bash
docker ps -a
docker start CONTAINER
docker exec -it CONTAINER sh
```

### 容器里没有 bash

错误：

```text
exec: "bash": executable file not found in $PATH
```

解决：

```bash
docker exec -it CONTAINER sh
```

### 命令不存在

错误：

```text
exec: "curl": executable file not found in $PATH
```

说明容器内没有安装 `curl`

解决方式：

Debian/Ubuntu：

```bash
docker exec -u root CONTAINER sh -c 'apt update && apt install -y curl'
```

Alpine：

```bash
docker exec -u root CONTAINER sh -c 'apk add --no-cache curl'
```

### 输入设备不是 TTY

错误：

```text
the input device is not a TTY
```

常见原因是在脚本、管道或重定向场景中使用了 `-t`

错误示例：

```bash
docker exec -it mysql-container mysql -uroot -p123456 mydb < backup.sql
```

正确示例：

```bash
docker exec -i mysql-container mysql -uroot -p123456 mydb < backup.sql
```

### 权限不足

错误：

```text
Permission denied
```

解决方式之一：

```bash
docker exec -u root -it CONTAINER sh
```

或：

```bash
docker exec -u root CONTAINER chmod +x /path/script.sh
```

注意不要随意改变生产容器中的文件权限

### 变量没有正确展开

错误示例：

```bash
docker exec my-app sh -c "echo $HOME"
```

这里 `$HOME` 可能被宿主机 Shell 先展开

更推荐：

```bash
docker exec my-app sh -c 'echo $HOME'
```

## 常用命令

进入容器：

```bash
docker exec -it CONTAINER bash
docker exec -it CONTAINER sh
```

以 root 进入：

```bash
docker exec -u root -it CONTAINER bash
```

指定目录进入：

```bash
docker exec -it -w /app CONTAINER bash
```

执行单条命令：

```bash
docker exec CONTAINER ls -lah /app
```

执行复杂命令：

```bash
docker exec CONTAINER sh -c 'cd /app && npm run build'
```

后台执行：

```bash
docker exec -d CONTAINER sh -c 'long-running-command'
```

设置临时环境变量：

```bash
docker exec -e APP_ENV=debug CONTAINER env
```

从文件读取环境变量：

```bash
docker exec --env-file .env CONTAINER env
```

导入 MySQL 数据：

```bash
docker exec -i mysql-container mysql -uroot -p123456 mydb < backup.sql
```

导出 MySQL 数据：

```bash
docker exec mysql-container mysqldump -uroot -p123456 mydb > backup.sql
```

## 实践

### 日常进入容器优先使用

```bash
docker exec -it CONTAINER sh
```

如果确定有 Bash：

```bash
docker exec -it CONTAINER bash
```

### 自动化脚本中尽量不用 `-t`

脚本中推荐：

```bash
docker exec CONTAINER command
```

或者需要输入时：

```bash
docker exec -i CONTAINER command
```

不推荐：

```bash
docker exec -it CONTAINER command
```

### 复杂命令使用 `sh -c`

推荐：

```bash
docker exec CONTAINER sh -c 'cd /app && command1 && command2'
```

### 注意容器内外路径区别

宿主机路径：

```bash
/home/user/project
```

容器内路径：

```bash
/app
```

`docker exec` 后面的路径都是**容器内部路径**。

例如：

```bash
docker exec my-app ls /app
```

这里 `/app` 是容器内的 `/app`

----

## 参考说明

<https://docs.docker.com/reference/cli/docker/container/exec/>

本文使用了生成式人工智能来协助整理
