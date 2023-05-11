##  一、基本概念

### 1、概述

- Docker 是一个用于开发、发布和运行应用程序的开放平台。

- Docker 提供了在称为容器的松散隔离环境中打包和运行应用程序的能力。

- 容器时轻量级的，包含运行程序所需的一切，无需依赖主机上的内容。允许在主机上同时运行多个容器。

### 2、Docker 架构

![Docker 架构图](https://docs.docker.com/engine/images/architecture.svg)

#### Docker Client（客户端）

Docker 客户端用于与 Docker 守护进程交互，客户端将命令发送到守护进程执行。客户端和守护进程使用 REST API，通过 UNIX 套接字或网络接口通信。客户端可以与多个守护进程通信，可以连接到本机或远程守护程序。

另一个 Docker 客户端是 DockerCompose，它允许使用由一组容器组成的应用程序。

#### Docker Deamon（守护程序）

Docker 守护程序负责构建、运行和分发 Docker 容器的繁重工作。 Docker 守护程序侦听 Docker API 请求并管理 Docker 对象（Images、Containers、Networks、Volumes）。守护进程还可以与其他守护进程通信以管理 Docker 服务。

#### Docker Registry（镜像仓库）

> 公共镜像仓库：[Docker Hub](https://hub.docker.com/) 

集中的存储、分发镜像的服务。

当使用 `docker pull` or `docker run` 命令时，将从配置的镜像仓库中提取所需的镜像。当使用 `docker push` 命令时，镜像像会被推送到配置的仓库中。 

#### Docker 对象

- 镜像：

  相当于是一个 `root` 文件系统，镜像名 = <仓库名>:<标签>。

  镜像是一个只读模板，其中包含创建 Docker 容器的说明。使用简单的语法创建一个 Dockerfile，定义创建和运行镜像所需的步骤。Dockerfile 中的每条指令都会在镜像中创建一个层。当更改 Dockerfile 并重建镜像时，仅重建哪些已更改的层。与其它虚拟化技术先比，这是使镜像如此轻量、小巧和快速的部分原因。

  运行容器时，它使用隔离的文件系统。此自定义文件系统由容器映像提供。由于镜像包含容器的文件系统，它必须包含运行应用程序所需的一切（所有依赖项、配置、脚本、二进制文件等）。镜像还包含容器的其他配置，例如环境变量、运行的默认命令、和其他元数据。 

- 容器：

  镜像是静态的定义， 容器是镜像的可运行实例。可以使用 Docker API 或 CLI 创建、启动、停止、删除、暂停容器。还可以将容器连接到一个或多个网络，将存储附加到它，甚至可以根据其当前状态创建新镜像。 

### 3、底层技术

Docker 是用 Go 语言编写，利用 Linux 内核的几个特性来提供其功能。Docker 使用一种称为容器命名空间的技术来提供隔离的工作空间。当运行容器时，Docker 会为该容器创建一组命名空间。这些命名空间提供了一层隔离，容器在各自的命名空间中运行，并且它的访问权限仅限于该命名空间。

### 4、容器与传统虚拟机对比

|        |  启动  | 硬盘使用  |   性能   |     系统支持量     |
| -----: | :----: | :-------: | :------: | :----------------: |
|   容器 |  秒级  | 一般为 MB | 接近原生 | 单机支持上千个容器 |
| 虚拟机 | 分钟级 | 一般为 GB |   弱于   |     一般几十个     |



## 二、安装 Docker

> 查询 Linux 内核版本：

```
cat /proc/version 
uname -a
uname -s
uname -m

# 查询 Linux 系统版本
cat /etc/redhat-release
```

### 1、使用 yum 安装

> CentOS 下安装

```powershell
# 安装依赖包
$ sudo yum install -y yum-utils 

# 配置 yum 软件源
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 官方源（国外）
# $ sudo yum-config-manager \
#     --add-repo \
#     https://download.docker.com/linux/centos/docker-ce.repo

# 安装 Docker 引擎
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

### 2、操作 Docker

```powershell
# 开机启动/禁用
$ systemctl enable docker
$ systemctl disable docker

# 启动 Docker
$ systemctl start docker

# 停止 Docker
$ systemctl stop docker

# 重启 Docker
$ systemctl restart docker

# 查看 Docker 概要信息
$ docker info

# 查看 Docker 帮助文档
$ docker --help
```

### 3、镜像加速器

```powershell
# 查看是否在 docker.service 文件中配置过镜像地址
$ systemctl cat docker | grep '\-\-registry\-mirror'

# 添加配置文件：/etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com", # 网易云加速器
    "https://mirror.baidubce.com"   # 百度云加速器
  ]
}

# 重启 Docker 服务
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker

# 如果从结果中看到了如下内容，说明配置成功
$ docker info
...
Registry Mirrors:
 https://hub-mirror.c.163.com/
...
```

### 4、Docker 安装完成

```powershell
$ docker run --rm hello-world
```

### 5、资源目录

> 与 Docker 相关的本地资源默认存放在 `/var/lib/docker/` 目录下：

```powershell
/var/lib/docker/
├─ buildkit		
├─ containers	容器
├─ image		镜像
├─ network		网络
├─ overlay2		
├─ plugins		
├─ runtimes		
├─ swarm		
├─ tmp			
├─ trust		
├─ volumes		数据卷
└─ ... 
```

### 6、卸载 Docker

```powershell
# 卸载 Docker Engine、CLI、Containerd 和 Docker Compose 软件包：
$ sudo yum remove docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 主机上的映像、容器、卷或自定义配置文件不会自动删除。要删除所有映像、容器和卷：
$ sudo rm -rf /var/lib/docker
$ sudo rm -rf /var/lib/containerd
```



## 三、使用镜像

### 1、搜索镜像

```
docker search <image_name>

	--limit 默认25
```

| NAME     | DESCRIPTION | STARS    | OFFICIAL | AUTOMATED      |
| -------- | ----------- | -------- | -------- | -------------- |
| 镜像名称 | 镜像说明    | 点赞数量 | 官方认证 | 是否是自动构建 |

### 2、下载镜像

```
# 标签：默认 latest
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

### 3、列出镜像

```
# 列出已下载到本地主机上的镜像
docker images <镜像名>
docker image ls <镜像名>

options：
	-a 列出本地所有镜像（中间层镜像）
	-q 只显示镜像ID
	
# 查看镜像、容器、数据卷所占空间
docker system df
```

| REPOSITORY | TAG          | IMAGE ID | CREATED  | SIZE     |
| ---------- | ------------ | -------- | -------- | -------- |
| 仓库名     | 标签（版本） | 镜像 ID  | 创建时间 | 镜像大小 |

### 4、镜像 Tag

```
# 为镜像创建新标签
docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

### 5、虚悬镜像

> 虚悬镜像：既没有仓库名，也没有标签的镜像。

```
# 显示虚悬镜像
docker image ls -f dangling=true

# 删除虚悬镜像
docker image prune
```

### 6、删除镜像

```
# 可以用镜像ID、镜像名删除镜像
docker image rm [选项] <镜像名 ...>

	-f 强制删除
	
简写：
docker rmi [选项] <镜像名 ...>
```

### 7、commit 定制镜像

> 不要使用 `docker commit` 定制镜像，定制镜像应该使用 `Dockerfile` 来完成。
>
> 镜像的分层：镜像是多层存储，每一层是在前一层的基础上进行的修改；而容器同样也是多层存储，是在以镜像为基础层，在其基础上加一层作为容器运行时的存储层。

```
# 将容器的存储层保存下来成为镜像
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]

OPTIONS:
	-a, --author  用户
	-m, --message 信息
	
# 查看容器的具体改动（存储层）
docker diff <容器ID或容器名>	

# 查看镜像的历史记录
docker history [镜像名]
```

### 8、镜像分层

```
# 查看镜像创建时各层的命令
docker image history <镜像名>

--no-trunc 不截断输出
```



## 四、操作容器

### 1、启动容器

```
docker run [OPTIONS] IMAGE [COMMAND][ARG...]

OPTIONS:
	--name <container_name>	指定容器名称
	-c, --cpu-shares		CPU份额（相对权重）
	-d, --detach			以守护进程运行
	-e, --env list			设置环境变量
	-h, --hostname			容器主机名
	-it						以交互式终端模式运行
	-m, --memory bytes		内存限制
	-p, --publish			将容器端口映射到主机，[host_port]:[container_port]
	-P, --publish-all		将 EXPOSE 暴露的端口映射到随机端口
	--rm					容器退出时自动移除容器（同时移除匿名数据卷）
	-v, --volume			绑定数据卷
	--volumes-from			从指定容器加载卷
	--mount					将文件系统挂载到容器
	-w, --workdir 			指定容器内的工作目录
    --privileged=true       授予此容器扩展权限（特权）
	-u                      -u 0 用root用户登录容器，而不是镜像的默认用户
    
COMMAND:
	env		查看镜像支持的环境变量
```

> 以下命令运行一个 `ubuntu` 容器，以交互方式附加到您的本地命令行会话，然后运行 `/bin/bash`
>

```
$ docker run -i -t ubuntu /bin/bash
```

当您运行此命令时，会发生以下情况（假设您使用的是默认注册表配置）：

- 如果您在本地没有`ubuntu`映像，Docker 会从您配置的注册表中提取它，就像您`docker pull ubuntu`手动运行一样。
- Docker 会创建一个新容器，就像您`docker container create` 手动运行命令一样。

- Docker 为容器分配一个读写文件系统，作为它的最后一层。这允许正在运行的容器在其本地文件系统中创建或修改文件和目录。

- Docker 创建了一个网络接口来将容器连接到默认网络，因为您没有指定任何网络选项。这包括为容器分配 IP 地址。默认情况下，容器可以使用主机的网络连接连接到外部网络。

- Docker 启动容器并执行`/bin/bash`. 因为容器以交互方式运行并附加到您的终端（由于`-i`and`-t` 标志），所以您可以在输出记录到终端时使用键盘提供输入。

- 当您键入`exit`终止`/bin/bash`命令时，容器会停止但不会被删除。您可以重新启动或删除它。

### 2、查看运行容器

```
docker ps [OPTIONS]
docker container ls

OPTIONS：
	-a 列出正在运行+历史运行（已停止）的容器
	-l 显示最近创建的容器
	-n 显示最近n个创建的容器
	-q 静默模式，只显示容器编号
```

### 3、终止容器

```
# 启动已停止运行的容器
docker start <容器ID/容器名>

# 重启容器
docker restart <容器ID/容器名>

# 停止容器
docker stop <容器ID/容器名>

# 强制停止容器
docker kill <容器ID/容器名>

# 删除已停止的容器
docker rm [OPTIONS] <容器ID/容器名 ...>
# 批量删除已停止的容器
docker container prune

OPTIONS：
	-f 强制删除，删除未停止的容器
	-v 删除容器时，同时删除数据卷
	
# 例：批量删除容器
docker rm -f $(docker ps -aq)
```

### 4、查看容器细节

```
# 获取容器的日志
docker logs [OPTIONS] <容器ID/容器名>

OPTIONS:
	-f 跟踪日志输出
	-n 显示条数，默认 all
	
# 显示容器的运行进程
docker top <容器ID/容器名>

# 返回 Docker 对象的底层信息
docker inspect [OPTIONS] <容器ID/容器名 ...>

# 查看具体项 -f,--format
docker inspect -f '{{json .State.Pid }}' <容器ID/容器名 ...> | jq

注：jq 可以对 json 文本进行格式化输出。
curl -O http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
yum install -y jq
```

### 5、进入容器

> 进入 -d 启动的守护状态容器。

```
# exit 会导致容器停止
docker attach [OPTIONS] <容器ID/容器名>

# exit 不会导致容器停止（推荐）
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

### 6、导入导出

```
# 拷贝容器文件到主机
docker cp 容器ID:容器内路径 目的主机路径

# 导出
docker export 容器ID > 文件名.tar

# 导入
cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号
docker import http://example.com/exampleimage.tgz example/imagerepo
```



## 五、使用 Docker 开发

> Dockfile 是用来构建 Docker 镜像的文本文件，是有一条条构建镜像所需的指令和参数构成的脚本。

### 1、Dockerfile 指令

##### FROM

> 继承基础镜像，当前镜像基于哪个镜像开始构建。

尽可能使用当前的官方图像作为图像的基础。我们推荐使用 `Alpine`，因为它受到严格控制且体积小（目前低于 6 MB），同时仍然是一个完整的 Linux 发行版。 

##### COPY & ADD

> COPY 将文件/目录从构建上下文目录中复制到新的一层的镜像内。
>

```
# 推荐
COPY [--chown=<user>:<group>] <src...> <dest>

OPTIONS:
	--chown=<user>:<group>	改变容器内文件的所属用户及所属组
	--from=<name>			从 from 指定的构建阶段中寻找源文件 <src>
	
注：
    - <src>  可以是多个，甚至可以是通配符。
    - <dest> 可以是镜像内的绝对路径，也可以是相对于工作目录的相对路径（WORKDIR 指定）。
    - <dest> 不存在时会自动创建。
    - 使用 COPY 指令，源文件的各种元数据（如读、写、执行权限、文件变更时间等）都会保留。
```

> ADD 指令和 COPY 的格式和性质基本一致。
>
> [Dockerfile 最佳实践文档](https://www.kancloud.cn/docker_practice/docker_practice/972883)：推荐使用 COPY 复制文件，仅在需要自动解压缩的场合使用 ADD。

```
# 仅在需要自动解压时使用
ADD [--chown=<user>:<group>] <源路径...> <目标路径>

OPTIONS:
	--chown=<user>:<group> 改变容器内文件的所属用户及所属组

注：
	- 当 <源路径> 是一个 URL 时，Docker 引擎会尝试下载这个文件放到 <目标路径> 去，下载的文件权限为 600，可以另外添加 RUN 层修改文件权限。
	- 当 <源路径> 是一个 tar 压缩文件时，压缩格式为 gzip、bzip2、xz 的情况下，ADD 指令将会自动解压缩后放到 <目标路径> 去。
	- ADD 指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。
```

##### RUN

> RUN 是在 `dicker build` 时运行。
>
> RUN 指令将在当前镜像上加新的一层，并执行任何命令和提交结果，生成的提交镜像将用于 Dockfile 中的后续步骤。（每一个 `RUN` 都是启动一个容器、执行命令、然后提交存储层文件变更。）
>
> 分层 RUN 指令和生成提交符合 Docker 核心概念，提交成本低，并且可以通过 docker history 中的任意步骤创建容器，像 git 代码控制一样。

```
# shell 格式
# 默认以 /bin/sh -c <command> 格式运行
# 等价于 RUN ["/bin/sh", "-c", "command"]
RUN <command>

# exec 格式（推荐）
# exec 形式会被解析为JSON格式，必须用双引号
RUN ["可执行文件", "参数1", "参数2", ...]
```

注：只有 RUN、COPY、ADD 指令会改变层，其他指令会创建临时中间图像，并且不会增加构建的大小。

##### CMD

> CMD 是在 `docker run` 时运行，可以有多个 CMD 指令，但只有最后一个生效，且会被 `docker run` 之后的参数覆盖。
>
> CMD 为容器指定默认要运行的程序及启动命令，程序运行结束，容器也就结束。

```
# shell 格式
# 等价于 CMD ["/bin/sh", "-c", "command"]
CMD <命令>

# exec 格式（推荐）
CMD ["可执行文件", "参数1", "参数2", ...]

# 参数列表格式
# 在指定 ENTRYPOINT 指令后，用 CMD 指定具体参数
CMD ["参数1", "参数2", ...]
```

##### ENTRYPOINT

> 类似于 CMD 指令，都是在指定容器启动程序及参数。但其不会被 `docker run` 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。 
>
> 在 docker run 时，也可以用 --entrypoint 选项来覆盖。
>
> 只能出现一次。

```
<ENTRYPOINT> "<CMD>"
ADD、COPY、ENV、EXPOSE、FROM、LABEL、USER、WORKDIR、VOLUME、STOPSIGNAL、ONBUILD、RUN
```

##### ENV

> 设置环境变量，ADD、COPY、ENV、EXPOSE、FROM、LABEL、USER、WORKDIR、VOLUME、STOPSIGNAL、ONBUILD、RUN 这些指令都支持环境变量的展开。
>
> 每 ENV 行创建一个新的中间层，就像 RUN 命令一样。 

```
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...

# 这样删除环境变量无效
RUN echo $ADMIN_USER > ./mark
RUN unset ADMIN_USER

# 在单个层中设置、使用和取消设置变量
RUN export ADMIN_USER="mark" \
    && echo $ADMIN_USER > ./mark \
    && unset ADMIN_USER
```

##### ARG

> 和 ENV 效果一眼，都是设置环境变量。
>
> 其默认值可以在 `docker build --build-arg 参数名=值` 时覆盖。
>
> 注：若 ARG 在 FROM 之前指定，则只能用于 FROM 中，要想在 FROM 之后使用，须再次指定。

```
ARG <参数名>[=<默认值>]
```

##### VOLUME

> 定义匿名卷
>
> 运行时，使用 `docker run -d -v mydata:/data xxx` 覆盖这个挂载设置。

```
VOLUME ["<路径1>", "<路径2>"...]
```

##### EXPOSE

> 声明运行时容器提供的服务端口（仅声明暂未开启）。
>
> 声明好处：
>
> - 便于理解这个镜像服务的守护端口，以方便映射；
> - 再运行时使用 `docker run -P` 时，会自动随机映射 EXPOSE 的端口。

```
EXPOSE <端口1> [<端口2>...]
```

##### WORKDIR

> 用来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，若目录不存在 WORKDIR 会帮你自动创建。

```
WORKDIR <工作目录路径>
```

##### USER

> 改变之后层的执行 `RUN`， `CMD` 以及 `ENTRYPOINT` 这类命令的身份。
>
> 注意，`USER` 只是帮助你切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。

```
USER <用户名>[:<用户组>]
```

##### HEALTHCHECK

> `HEALTHCHECK` 只可以出现一次，如果写了多个，只有最后一个生效。 

```
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

选项：
    --interval=<间隔>：两次健康检查的间隔，默认为 30 秒；
    --timeout=<时长>：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
    --retries=<次数>：当连续失败指定次数后，则将容器状态视为 unhealthy，默认 3 次。
```

##### ONBUILD

> `ONBUILD` 是一个特殊的指令，它后面跟的是其它指令，比如 `RUN`, `COPY` 等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。

```
ONBUILD <其它指令>
```

##### LABEL

> 用来给镜像以键值对的形式添加一些元数据。

```
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

##### SHELL

> SHELL 指令可以指定 RUN ENTRYPOINT CMD 指令的 shell，Linux 中默认为 `["/bin/sh", "-c"]`。

```
SHELL ["executable", "parameters"]
```

### 2、从 Dockerfile 构建镜像（docker build）

> Docker 能够通过 stdin 管道传输 Dockerfile 使用本地或远程构建上下文来构建镜像。

```
docker build [OPTIONS] PATH | URL | -

OPTIONS:
	-t,--tag	镜像名称
	-f			Dockerfile 的名称（默认 'PATH/Dockerfile'）
	--no-cache	不使用缓存
	
PARAMETERS:
	PATH	使用本地（PATH）构建上下文中的 Dockerfile 构建镜像
	URL		使用远程（URL）构建上下文中的 Dockerfile 构建镜像
	-		通过 stdin 管道传输 Dockerfile 构建镜像
			用 `-` 作为文件名来指示 Docker 从 stdin 读取 Dockerfile
```

> Docker 的工作原理（了解构建上下文）

- Docker Client 将构建命令后面指定的 PATH (`.`) 目录下的所有文件打包成一个 tar 包，发送给 Docker Deamon；

- Docker Deamon 收到客户端发送的 tar 包后解压（得到：构建上下文），根据 Dockerfile 里面的指令进行镜像的分层构建；

- 如果不指定 Dockerfile，客户端默认在指定上下文路径下寻找 Dockerfile 文件；Dockerfile 可以不在构建上下文路径下，此时需要通过 -f 参数明确指定。

#### （1）使用 PATH 构建上下文中的 Dockerfile 构建镜像

```
# 将当前目录作为构建上下文，并指定镜像名称
# 默认用 ./Dockerfile 构建镜像
docker build -t app:v1 .
```

#### （2）通过 stdin 中的 Dockerfile 构建映像，而不发送构建上下文（仅包含 Dockerfile）

> 通过 stdin 管道传输 Dockerfile 对于在不将 Dockerfle 写入磁盘的情况下执行一次性构建非常有用，或者在生成 Dockerfil 之后不应继续存在的情况下也非常有用。

```
docker build -t app:v2 -<<EOF
FROM alpine
RUN echo "hello world"
EOF

# 如果使用此语法构建，使用 COPY 或 ADD 的 Dockerfile 将失败。
docker build -t test -<<EOF
FROM alpine
COPY main.go /
RUN echo "hello world"
EOF

Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM alpine
 ---> 9c6f07244728
Step 2/3 : COPY main.go /
COPY failed: file not found in build context or excluded by .dockerignore: stat main.go: file does not exist
```

#### （3）使用 stdin 中的 Dockerfile 从本地构建上下文构建镜像

```
docker build -t app:v3 -f- . <<EOF
FROM golang:1.17-alpine
COPY main.go /
RUN go run /main.go
EOF

Sending build context to Docker daemon  5.167kB
Step 1/3 : FROM golang:1.17-alpine
 ---> 270c4f58750f
Step 2/3 : COPY main.go /
 ---> 3330649f8230
Step 3/3 : RUN go run /main.go
 ---> Running in 15793f4c0842
Hello YCZ!
Removing intermediate container 15793f4c0842
 ---> 2fdeff45a2e8
Successfully built 2fdeff45a2e8
Successfully tagged app:v3
```

#### （4）使用 stdin 中的 Dockerfile 从远程构建上下文构建镜像

```
docker build -t app:v4 -f- https://github.com/docker-library/hello-world.git <<EOF
FROM busybox
COPY hello.c ./
EOF

# Docker 会在本机执行 git clone，并将这些文件作为构建上下文发送给守护进程
# 因此需要再主机上安装 git
```

#### （5）使用 .dockerignore 排除

- 使用 Dockfile 创建镜像时要添加 `.dockerignore` 文件或使用干净的工作目录。
- `.dockerignore` 需要在构建上下文的根目录中。

### 3、案例

```
# 目录结构

/app
├─ Dockerfile
├─ main.go
└─ ...
```

#### （1）创建 Dockerfile

> Dockerfile

```dockerfile
# syntax=docker/dockerfile:1 
# 可选，指示 Docker 构建器在解析 Dockerfile 时使用什么语法

# FROM 继承
# 告诉 Docker 我们的镜像中包含 golang:1.17-alpine 镜像的所有功能
FROM golang:1.17-alpine

# 工作目录
WORKDIR /app

# 将当前目录复制到镜像中的 /app
COPY . .

# 执行 go build
RUN go build -o server main.go

# CMD 可执行程序必须为绝对路径
CMD ["/app/server"]
```

> main.go

```go
package main

import (
    "fmt"
)

func main() {
    fmt.Println("Hello YCZ!")
}
```

#### （2）构建镜像并运行

```
# 构建
> cd /app
> docker build -t hello-ycz:v1 .

# 查看镜像（发现生成的镜像过于庞大：316MB）
> docker images
REPOSITORY    TAG           IMAGE ID       CREATED          SIZE
hello-ycz     v1            98fa0a4e5ab9   33 minutes ago   316MB（太大）

# 运行镜像
docker run --name=hello-ycz-v1 hello-ycz:v1
```

#### （3）多阶段构建（镜像小）

> MultistageDockerfile

```dockerfile
# syntax=docker/dockerfile:1

## Build
# 第一构建阶段：仅用于 build 出可执行文件
# AS <NAME> 命名构建阶段
FROM golang:1.17-alpine AS build

WORKDIR /app

COPY . .

RUN go build -o server main.go

## Deploy
# 第二构建阶段：这里的改变将影响生成的镜像
# 轻量的 alpine：适用于静态二进制文件的精益部署
FROM alpine 

WORKDIR /

# 从 build 阶段复制 server
# 该文件仅存在于前一个 Docker 阶段，因此需要用 --from=build 来复制它
COPY --from=build /app/server /server

CMD ["/server"]
```

> 构建 & 运行

```
# 构建
> docker build -t hello-ycz -f MultistageDockerfile .

# 查看镜像（此时镜像：7.32MB）
> docker images
REPOSITORY    TAG           IMAGE ID       CREATED          SIZE
hello-ycz     latest        5f7d56bf57ad   54 seconds ago   7.32MB（小）
hello-ycz     v1            98fa0a4e5ab9   40 minutes ago   316MB

# 运行镜像
> docker run --name=hello-ycz hello-ycz
```

### 4、BuildKit 构建镜像

```
# 启用 BuildKit 构建
DOCKER_BUILDKIT=1 docker build .

# 写入配置文件 `/etc/docker/daemon.json`
{
	"features": { "buildkit": true } 
}
```



## 六、Docker 网络 - 待完善

### 1、基本命令

```
docker network COMMAND

Commands：
    connect     将容器连接到网络
    create      创建网络
    disconnect  断开容器与网络的连接
    inspect     显示一个或多个网络上的详细信息
    ls          列出网络
    prune       删除所有未使用的网络
    rm          删除一个或多个网络
```

### 2、网络模式

| 网络模式  | 说明                                                         | 配置                     |
| --------- | ------------------------------------------------------------ | ------------------------ |
| bridge    | 虚拟网桥（默认模式），为每个容器分配、设置 IP，并将容器连接到 docker0。 | --network bridge         |
| host      | 容器将不会虚拟出自己的网卡、配置自己的 IP 等，而是使用宿主机的 IP 和端口。 | --network host           |
| none      | 容器有独立的网络命名空间，但并没有对其进行任何网络设置，如分配 veth pair 和网桥连接、IP 等。 | --network none           |
| container | 新创建的容器不会创建自己的网卡和配置自己的IP，而是和一个指定的容器共享 IP，端口范围等。 | --network container:NAME |

#### （1）bridge

1、整个宿主机的网桥模式都是 docker0，每个接口叫 veth，在本地主机和容器内分别创建一个虚拟接口，并让他们彼此联通（这一对接口叫 veth pair）；

2、每个容器实例内部也有一块网卡，每个接口叫 eth0；

#### （2）host

```
docker run -d --network host redis

# host 模式下 -p 配置无效
```

#### （3）none

#### （4）container

> 当父容器停掉后，子容器的网络不可用。

#### （5）自定义



## 七、Docker 数据管理

### 1、管理数据卷

> 数据卷是被设计用来持久化容器数据的。

① 数据卷可以在容器之间共享和重用；

② 对数据卷的修改会立马生效；

③ 对数据卷的更新，不会影响镜像；

```
# 创建数据卷
docker volume create <volume_name>

# 查看所有数据卷
docker volume ls

# 查看指定数据卷的信息
docker volume inspect <volume_name>
[
    {
        "CreatedAt": "2022-08-18T00:20:53+08:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/test/_data",
        "Name": "test",
        "Options": {},
        "Scope": "local"
    }
]


# 删除数据卷
docker volume rm <volume_name>

# 清理未使用的数据卷
docker volume prune
```

### 2、挂载数据卷

> 卷对于备份、恢复和迁移非常有用。

```
docker run -d --name <容器名> \
	# 方式1：
	-v <volume_name>:<容器中挂载的路径>:[rw, readonly|ro] \
	# 方式2：
	--mount source=<volume_name>,target=<容器中挂载的路径>,[readonly|ro] \
	<镜像名>
	
	# 注：如遇权限不足
    --privileged=true  # 授予此容器扩展权限
    
# 匿名卷挂载到容器路径
-v /var/lib/mysql
```

注：

- 若不指定挂载卷，则使用匿名卷（随机 hash 值名）。使用 `--rm` 选项容器退出时，自动移除容器和卷。
- 若指定卷不存在会自动创建。
- 若指定卷已存在，则会自动加载卷中数据到容器中（启动MySQL时，卷不能被占用）。

> 启动一个挂载卷的容器

```
docker run -d --name mysql-3306 \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=root \
    -v database:/var/lib/mysql \
    mysql:5.7
    
# 查看容器的挂载信息
docker inspect -f '{{json .Mounts}}' mysql-3306 | jq
[
  {
    "Type": "volume",
    "Name": "database",
    "Source": "/var/lib/docker/volumes/database/_data",
    "Destination": "/var/lib/mysql",
    "Driver": "local",
    "Mode": "z",
    "RW": true,
    "Propagation": ""
  }
]
```

> 从指定容器加载卷（共享容器数据卷）

```
docker run -d --name mysql-3307 \
    -p 3307:3306 \
    -e MYSQL_ROOT_PASSWORD=root \
    --volumes-from mysql-3306 \
    mysql:5.7
```

### 3、绑定文件系统

```
docker run -d --name <容器名> \
	# 方式1：
	-v <宿主机路径>:<容器中挂载的路径>:[rw, readonly|ro, rprivate] \
	# 方式2：
	--mount source=<宿主机路径>,target=<容器中挂载的路径>,[readonly|ro] \3
	<镜像名>
	
	# 注：如遇权限不足
    --privileged=true  # 授予此容器扩展权限
    
# 格式：--mount <key>=<value>
	<key>					<value>
	type					bind|volume|tmpfs
	source|src				主机路径
	destination|dst|target	容器中的绝对路径
	readonly				绑定挂载以只读方式挂载到容器中
	bind-propagation		绑定传播属性：rprivate, private, rshared, shared, rslave, slave
```

注：

- 使用 -v 绑定文件系统时，主机路径不存在，会自动创建目录。

- 使用 --mount 绑定文件系统时，主机路径不存在会产生错误。

**绑定传播属性：**

| 传播设置   | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| `rprivate` | 默认值。与 private 相同，这意味着原始挂载点或副本挂载点中的任何挂载点都不会向任一方向传播。 |
| `private`  | 挂载是私有的。其中的子挂载不暴露给副本挂载，副本挂载的子挂载不暴露给原始挂载。 |
| `rshared`  | 与 shared 相同，但传播也延伸到嵌套在任何原始或副本挂载点内的挂载点。 |
| `shared`   | 原始挂载的子挂载暴露给副本挂载，并且副本挂载的子挂载也传播到原始挂载。 |
| `rslave`   | 与 slave 相同，但传播也延伸到嵌套在任何原始或副本挂载点内的挂载点。 |
| `slave`    | 类似于 shared 挂载，但仅限于一个方向。如果原始挂载暴露了子挂载，则副本挂载可以看到它。但是，如果副本挂载暴露了子挂载，则原始挂载无法看到它。 |

> 启动一个绑定文件系统的容器

```
docker run -d --name mysql-3308 \
    -p 3308:3306 \
    -e MYSQL_ROOT_PASSWORD=root \
    -v /data/mysql:/var/lib/mysql \
    mysql:5.7
    
# 查看绑定信息
docker inspect -f '{{json .Mounts}}' mysql-3308 | jq
[
  {
    "Type": "bind",
    "Source": "/data/mysql",
    "Destination": "/var/lib/mysql",
    "Mode": "",
    "RW": true,
    "Propagation": "rprivate"
  }
]
```

### 4、tmpfs 挂载





## 八、Docker Compose

> `Compose` 是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。 
>
> `Compose` 是一个用于定义和运行多个 Docker 容器的工具。
>
> 通过一个单独的 `docker-compose.yml` 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。
>
> - 服务 (`service`)：一个应用容器，实际上可以运行多个相同镜像的实例。
> - 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元。

### 1、安装

```
## 1 安装为可执行程序（docker-compose）
# 下载
$ curl -L https://github.com/docker/compose/releases/download/v2.7.0/docker-compose-linux-x86_64 > /usr/local/bin/docker-compose

# 可执行权限
$ chmod +x /usr/local/bin/docker-compose

# 卸载
$ sudo rm /usr/local/bin/docker-compose

# 安装完成
$ docker-compose version

## 2 安装为 Docker 插件（docker compose）
$ cp /usr/local/bin/docker-compose /usr/local/lib/docker/cli-plugins

# 安装完成
$ docker-compose version
```

### 2、docker-compose 模板

> 注意：不能使用 tab 缩进，只能用空格。

```yml
# 模板版本
version: "3.7"

# 项目名称（默认：directory name）
name: "app"

# 定义服务（容器）
services:
    # 第一个服务
    web:
        # 容器名
        container_name: web
        
        # 镜像名（TAG 未设置或为空时，使用 latest）
        image: "web:${TAG:-latest}"  
        
        # 网络配置（数组）
        networks:                    
            - web-network
            
        # 端口映射（数组）
        ports:                       
            - 8080:80
            
        # 数据挂载（数组）
        # 同时配置挂载、绑定到同一目录，后面的配置会覆盖前面的，后面的生效
        volumes:                     
            - web-data:/app    # 挂载数据卷
        #   - /data/web:/app   # 文件系统挂载
        
        # 命令行
        command: "bash"
        
        # 在容器中设置环境变量
        environment:                 
            - TAG=v1   # docker run -e VARIABLE=VALUE ...
            - DEBUG    # docker run -e VARIABLE ... 将环境变量从 shell 直接传递给容器
        
        # 将环境变量从外部文件传递到容器（默认从：./.env）
        env_file:                    
            - .env     # docker run --env-file=FILE ...

    # 第二个服务
    
    # ...
 
 
# 定义数据卷
# docker-compose 文件中绑定未定义的数据卷时，不会自动创建，需手动创建
volumes:
    web-data:
    
# 定义网络
networks:
    web-network:
```

> 环境变量：

- `${TAG:-latest}`：如果环境变量在环境中未设置或为空，则使用默认值。
- `${TAG-latest}`：如果环境变量在环境中未设置，则使用默认值。
- `${TAG:?latest}`：如果环境变量在环境中未设置或为空，则退出，并显示 err 信息。
- `${TAG?latest}`：如果环境变量在环境中未设置，则退出，并显示 err 信息。

- 环境变量的优先级：？？？

  - .env 文件 > docker-compose 文件
  
  docker-compose 文件 > 命令行参数 > .env 文件 > Dockerfile > 未定义变量

> 环境文件：

- Compose 期望 `.env` 文件中的每一行都具有 `KEY=VALUE` 格式 。
- 引号没有特殊处理。这意味着它们是 VALUE 的一部分。所以字符串不用加 `""`。

### 3、命令详解

> 如无特殊说明，命令的对象是项目，项目中所有的服务都会受命令影响。

```
# 基本格式
docker compose [OPTIONS] COMMAND

Options:
	--env-file PATH             指定替代环境文件
	-f, --file FILE             指定使用的 Compose 模板文件
                                (default: ./docker-compose.yml)
    --project-directory string  指定项目目录
                                （默认：第一个指定 compose 文件的路径）
	-p, --project-name NAME     指定项目名称
                                (default: directory name)

Commands:

# 生成或重建服务
docker-compose build [SERVICE...]

# 将 compose 文件转换为平台的规范格式并打印
docker-compose convert

# 
```

```

```

（2）



### 4、案例

#### （1）Dockerfile

> 用 Dockerfile 定义应用程序的环境，以便可以在任何地方复制它。

```
# Dockerfile

...
```

#### （2）docker-compose.yml

> 在 `docker-compose.yml` 中定义组成应用程序的服务，以便它们可以在隔离环境中一起运行。

```dockerfile
    # 第二个服务
    database:
        container_name: mysql-3306
        image: mysql:5.7
        ports:
            - 3306:3306
        volumes:
            - /data/mysql:/var/lib/mysql
        environment:
            - MYSQL_ROOT_PASSWORD=root

    # 第三个服务
    cache:
        container_name: redis
        image: redis
        ports: 
            - 6379:6379
        volumes: 
            - /data/redis:/data

```

#### （3）docker compose up

> `docker compose up`  启动并运行您的整个应用程序。

```
docker-compose up -d
```

注意：

- `docker run` 时，会自动创建不存在的数据卷，但在 composer 中不会自动创建，需要在顶层 volumes 部分定义数据卷，然后才能在服务中挂载数据卷。



## 十、其它

### 1、访问仓库





### 3、Portainer Docker 可视化工具





### 4、CIG 容器重量级监控系统

CAdvisor：容器资源监控工具，包括 内存、CPU、网络IO、磁盘IO等。

InfluxDB：Go编写的一个开源分布式时序、事件和指标数据库。

Granfana：数据监控分析可视化平台。









