# docker-notes

非常实用的 docker 笔记，灰常滴推荐呀

## 基础概念

### 镜像

镜像是 docker 的基础元素，我们从网上使用 `docker pull` 命令下载下来的东西就叫做镜像

### 容器

容器都是由镜像创建出来的，当我们使用 `docker run` 命令的时候，docker 自动就会帮我们创建一个容器，每个容器都是一个独立的 linux 系统，不过可能是不同的 linux 发行版。一个镜像可以创建多个不同名字的容器，但是一般都是一对一的关系，因为没人莫名其妙会去创建多个容器玩

## 基础操作

### 寻找镜像

大家可以到 [Docker Hub](https://hub.docker.com/search?q=) 来寻找所有公开的镜像

### 下载镜像

``` bash
docker pull xxx
```

最常用的 docker 镜像：alpine ，alpine 是一个非常轻量的 linux 发行版，所以很多镜像都以这个作为基础，打包出来的镜像体积会很小

所以下面都以

``` bash
docker pull alpine
```

为基础

### 查看所有镜像

``` bash
docker images
```

如果你刚刚执行过上面的 pull 命令的话，那么你就可以在控制台看到你刚刚下载的镜像

### 创建一个容器并运行

``` bash
docker run --name c01 alpine
```

|参数|说明|备注|
|---|---|---|
|run|从镜像创建一个容器并运行||
|--name c01|自定义容器名为 c01|如果没有 --name 参数则 docker 会取一个随机名字|
|alpine|代表从 alpine 镜像创建容器||

但是我们运行之后发现，容器一启动就停止了，这是因为 docker 的默认策略就是如果容器内没有任务就会自动停止，所以如果你在学习阶段想让某个容器持续运行，那么可以使用

``` bash
docker run -d --name c02 alpine tail -f /dev/null
```

|参数|说明|备注|
|---|---|---|
|-d|使容器在后台运行||
|tail -f /dev/null|代表在容器内运行的命令|该命令是一种让容器持续运行的常用手段|

打开容器的终端：

``` bash
docker exec -it c02 sh
```

|参数|说明|备注|
|---|---|---|
|-it|是 -i -t 参数的结合体||
|-i|表示将容器的标准输入（stdin）保持打开状态。这允许你与容器进行交互，例如通过键盘输入命令或数据||
|-t|表示为容器分配一个伪终端（pseudo-TTY）。这会模拟一个终端设备，使得容器内的命令输出能够以易读的方式显示在终端上||
|exec|在容器中执行某条命令||
|sh|打开终端||

从容器内退出：

``` bash
exit
```

创建一个容器并打开它的终端

``` bash
docker run -it --name c02 alpine sh
```

### 停止容器

``` bash
docker stop c02
```

### 删除容器

#### 强制删除单个容器

``` bash
docker rm -f c02
```

|参数|说明|备注|
|---|---|---|
|-f|代表强制删除，即使目标还在运行中||

#### 强制删除所有容器

``` bash
docker rm -f $(docker ps -aq)
```

### 删除镜像

#### 强制删除单个镜像

``` bash
docker rmi -f alpine
```

#### 强制删除所有镜像

``` bash
docker rmi -f $(docker images -aq)
```

Windows 请使用 Windows PowerShell，CMD 无法运行此命令

### 查看镜像内容

有时候我们只想提前看看容器里有什么，而不想手动创建再手动删除，那么可以使用命令

``` bash
docker run -it --rm alpine
```

|参数|说明|备注|
|---|---|---|
|--rm|在退出容器后删除该容器||

## Dockerfile

当我们想将自己的项目在 docker 内运行的时候，就需要用到 dockerfile 了，通过编写 dockerfile 文件，我们可以把本地文件拷贝到容器内部，再在 dockerfile 文件里写上项目运行所需要执行的命令，编译
打包运行什么的，那么其它人就可以通过运行这个 dockerfile 文件来快速运行一个一模一样的项目

### 运行流程

简单说明一下 Dockerfile 文件的运行流程

Dockerfile 的运行其实分为通过 Dockerfile 构建镜像，与运行构建出来的镜像两步

``` dockerfile
FROM maven:3.9.4-eclipse-temurin-17-alpine

WORKDIR /app

COPY . /app

RUN cp ./maven-settings/settings.xml /usr/share/maven/conf/settings.xml \
    && mvn package -P docker

CMD ["java", "-jar", "./target/backend-0.0.1-SNAPSHOT.jar"]
```

这是一个简单的 Dockerfile 文件，当我们通过该文件构建镜像时，docker 会先自动下载 FROM 行内指定的镜像，并自动创建一个该镜像的临时容器，接着运行剩余的命令（除了 CMD 行），最后将临时容器的最终状态打包成一个镜像并存储

当我们通过构建出来的镜像创建一个容器并运行时，容器则会自动执行 CMD 行的命令，一般也就是启动项目所需的命令

### 新建 Dockerfile

我们一般会在项目的根目录下新建一个 Dockerfile 文件 （无后缀名，按照规范 D 应该为大写）

本项目内置了一个简单的 Vue 项目，大家可以在 vue-demo 内新建一个 Dockerfile 来练手

### 编写 Dockerfile

#### 选择基础镜像

打开 Dockerfile 文件，首先我们需要使用 `FROM` 命令来设置一个基础镜像，可以是上文提到的 alpine，也可以是其它的具体镜像，例如 Node，Mysql，Java，看你具体的需求

因为我们是部署 Vue 项目，所以显然选择一个 Node 镜像是最合适的，所以我们在第一行写入

``` dockerfile
FROM node:lts-alpine3.18
```

#### 设置工作目录

``` dockerfile
WORKDIR /app
```

临时容器的默认路径都是在根目录，如果直接使用会显得很乱，上述命令会在临时容器内创建一个名为 app 的文件夹，作为临时容器的工作目录

#### 复制本地文件

``` dockerfile
COPY . /app
```

该命令会自动在临时容器内创建一个名为 app 的文件夹，并复制当前目录下的所有文件到临时容器内的 app 文件夹内

#### 设置忽略文件

我们在本地测试的时候，当前目录可能会存在依赖库文件，例如 node_modules，如果把这个也复制到临时容器里显然是不合适的，不仅会影响部署速度，还会影响后续命令的执行。我们可以通过编写 .dockerignore 文件来让 dockerfile 执行操作时忽略某些文件，例如在 .dockerignore 内写入 `/node_modules/` 就可以让复制文件的时候忽略 node_modules 文件夹

#### 编写部署命令

使用 `RUN` 命令可以在临时容器内执行对应的命令

``` dockerfile
RUN npm install pnpm -g
```

该命令可以在临时容器内安装 pnpm

``` dockerfile
RUN pnpm install
```

该命令可以安装 Vue 项目所需的依赖库

但是根据菜鸟教程的说法，分行命令会造成额外的损失，所以如果有连续的命令，更推荐使用 `\` 来组合，例如把上面的两段命令改为

``` dockerfile
RUN npm install pnpm -g \
    && pnpm install
```

#### 编写运行命令

使用 `CMD` 命令可以在每次运行容器时都执行一次该命令，一般也就是项目启动所需的命令

``` dockerfile
CMD pnpm run dev
```

#### 通过 Dockerfile 构建镜像

在进行这一步之前，强烈推荐你先到 Docker Hub 注册一个账号，因为等下会用到账号的用户名

构建命令

``` bash
docker build -t blingyshs/my-vue-project:0.0.1 .
```

|参数|说明|备注|
|---|---|---|
|-t|代表给容器打标签，加上就对了||
|blingyshs/|你自己的 Docker Hub 用户名+一个/|不加也可以，只是无法将自己的镜像上传到 Docker Hub 而已，不影响本地使用|
|my-vue-project|自己给镜像取的名字，随便取||
|:0.0.1|冒号用于分隔镜像名与标签，标签没有固定格式，随便写，这里是 0.0.1||
|.|代表当前目录，不能省略||

#### 运行 Dockerfile 构建出来的镜像

``` bash
docker run --name mvp01 -p 5173:5173 blingyshs/my-vue-project:0.0.1
```

|参数|说明|备注|
|---|---|---|
|-p|映射端口|-p 5173:5173 代表映射容器内部的 5173 端口到本地的 5173 端口，两个端口不一定要一致|

## Docker Compose

下次再写
