#	简介

为了方便docker管理，需要一套工具来管理docker。现在出现的管理工具很多，主要介绍三种管理工具：compose、swarm和kubernetes

Compose 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。

Compose 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 Compose 来进行编排管理。


#   Compose

现在大部分开源docker都是用Compose来制定docker镜像和服务

这个和ansible-playbook很相似,把需要构建的镜像、服务和启动镜像的命令全部写成yml文件。

Compose 中有两个重要的概念：

*   服务（service）：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。

*   项目(project)：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

##  安装

使用yum安装

    yum install epel-release


    yum install -y python-pip


    pip install docker-compose


    yum upgrade python*
    
测试是否安装成功

    docker-compose -v
    
显示以下信息就说明是安装成功    
    
```
docker-compose version 1.14.0, build c7bdf9e
```

##  使用


对于 Compose 来说，大部分命令的对象既可以是项目本身，也可以指定为项目中的服务或者容器。如果没有特别的说明，命令对象将是项目，这意味着项目中所有的服务都会受到命令影响。

有两种使用方式，一种是命令行模式一种是yml文件方式，和ansible很像。



执行 docker-compose [COMMAND] --help 或者 docker-compose help [COMMAND] 可以查看具体某个命令的使用格式。

### 命令行

Compose 命令的基本的使用格式是

    docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
    
options：

-f, --file FILE 指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定。

-p, --project-name NAME 指定项目名称，默认将使用所在目录名称作为项目名。

--x-networking 使用 Docker 的可拔插网络后端特性（需要 Docker 1.9 及以后版本）。

--x-network-driver DRIVER 指定网络后端的驱动，默认为 bridge（需要 Docker 1.9 及以后版本）。

--verbose 输出更多调试信息。

-v, --version 打印版本并退出。


COMMAND:

列举出常见的命令

*   build

构建（重新构建）项目中的服务容器

格式为：

    docker-compose build [options] [SERVICE...]
    
服务容器一旦构建后，将会带上一个标记名，例如对于 web 项目中的一个 db 容器，可能是 web_db

可以随时在项目目录下运行 docker-compose build 来重新构建服务


选项包括：

--force-rm 删除构建过程中的临时容器。
--no-cache 构建镜像过程中不使用 cache（这将加长构建过程）。
--pull 始终尝试通过 pull 来获取更新版本的镜像。

*   kill

通过发送 SIGKILL 信号来强制停止服务容器。

格式：

    docker-compose kill [options] [SERVICE...]
    
支持通过 -s 参数来指定发送的信号，例如通过如下指令发送 SIGINT 信号

    docker-compose kill -s SIGINT
    
*   logs

查看服务容器的输出。默认情况下，docker-compose 将对不同的服务输出使用不同的颜色来区分。可以通过 --no-color 来关闭颜色  

格式：

    docker-compose logs [options] [SERVICE...]


    
*   port

打印某个容器端口所映射的公共端口

格式：

    docker-compose port [options] SERVICE PRIVATE_PORT
    
选项：

--protocol=proto 指定端口协议，tcp（默认值）或者 udp。
--index=index 如果同一服务存在多个容器，指定命令对象容器的序号（默认为 1）。    
    
*   ps

列出项目中目前的所有容器

格式：

    docker-compose ps [options] [SERVICE...]
    
选项：

-q 只打印容器的 ID 信息。

*   pull

拉取服务依赖的镜像

格式：

    docker-compose pull [options] [SERVICE...]
    
选项：

--ignore-pull-failures 忽略拉取镜像过程中的错误


*   start

启动已经存在的服务容器

格式：

    docker-compose start [SERVICE...]
    
*   stop

停止已经处于运行状态的容器，但不删除它。通过 docker-compose start 可以再次启动这些容器

格式：

    docker-compose stop [options] [SERVICE...]
    


*   pause

暂停一个服务容器

格式：

    docker-compose pause [SERVICE...]
    
*   unpause

恢复处于暂停状态中的服务

格式：

    docker-compose unpause [SERVICE...]
    

*   restart

重启项目中的服务

格式：

    docker-compose restart [options] [SERVICE...]

选项：

-t, --timeout TIMEOUT 指定重启前停止容器的超时（默认为 10 秒）

*   rm

删除所有（停止状态的）服务容器。推荐先执行 docker-compose stop 命令来停止容器

格式：

    docker-compose rm [options] [SERVICE...]
    
选项：

-f, --force 强制直接删除，包括非停止状态的容器。一般尽量不要使用该选项。
-v 删除容器所挂载的数据卷。

*   run

在指定服务上执行一个命令，默认情况下，如果存在关联，则所有关联的服务将会自动被启动，除非这些服务已经在运行中。

该命令类似启动容器后运行指定的命令，相关卷、链接等等都将会按照配置自动创建。

两个不同点：

给定命令将会覆盖原有的自动运行命令；
不会自动创建端口，以避免冲突。
如果不希望自动启动关联的容器，可以使用 --no-deps 选项

格式：

    docker-compose run [options] [-p PORT...] [-e KEY=VAL...] SERVICE [COMMAND] [ARGS...]

选项：

-d 后台运行容器。

--name NAME 为容器指定一个名字。

--entrypoint CMD 覆盖默认的容器启动指令。

-e KEY=VAL 设置环境变量值，可多次使用选项来设置多个环境变量。

-u, --user="" 指定运行容器的用户名或者 uid。

--no-deps 不自动启动关联的服务容器。

--rm 运行命令后自动删除容器，d 模式下将忽略。

-p, --publish=[] 映射容器端口到本地主机。

--service-ports 配置服务端口并映射到本地主机。

-T 不分配伪 tty，意味着依赖 tty 的指令将无法运行。

列子：

启动一个 ubuntu 服务容器，并执行 ping docker.com 命令

    docker-compose run ubuntu ping docker.com
    

*   scale

设置指定服务运行的容器个数，一般的，当指定数目多于该服务当前实际运行容器，将新创建并启动容器；反之，将停止容器。

格式：

    docker-compose scale [options] [SERVICE=NUM...]
    
选项：

-t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）

列子：

启动3个web容器，2个db容器

    docker-compose scale web=3 db=2
    

*   up

该命令十分强大，它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作

链接的服务都将会被自动启动，除非已经处于运行状态

可以说，大部分时候都可以直接通过该命令来启动一个项目

默认情况，docker-compose up 启动的容器都在前台，控制台将会同时打印所有容器的输出信息，可以很方便进行调试

当通过 Ctrl-C 停止命令时，所有容器将会停止

如果使用 docker-compose up -d，将会在后台启动并运行所有的容器。一般推荐生产环境下使用该选项

默认情况，如果服务容器已经存在，docker-compose up 将会尝试停止容器，然后重新创建（保持使用 volumes-from 挂载的卷），以保证新启动的服务匹配 docker-compose.yml 文件的最新内容。如果用户不希望容器被停止并重新创建，可以使用 docker-compose up --no-recreate。这样将只会启动处于停止状态的容器，而忽略已经运行的服务。如果用户只想重新部署某个服务，可以使用 docker-compose up --no-deps -d <SERVICE_NAME> 来重新创建服务并后台停止旧服务，启动新服务，并不会影响到其所依赖的服务

格式：

    docker-compose up [options] [SERVICE...]


选项：

-d 在后台运行服务容器。

--no-color 不使用颜色来区分不同的服务的控制台输出。

--no-deps 不启动服务所链接的容器。

--force-recreate 强制重新创建容器，不能与 --no-recreate 同时使用。

--no-recreate 如果容器已经存在了，则不重新创建，不能与 --force-recreate 同时使用。

--no-build 不自动构建缺失的服务镜像。
     
-t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）。


### yml文件

模板文件是使用 Compose 的核心，指令关键字也比较多，其实和命令行差不多,只是把参数写成完整形式

<pre>

version: "2"
services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"

</pre>

这是创建一个服务的基本格式

每个服务都必须通过 image 指令指定镜像或 build 指令（需要 Dockerfile）等来自动构建生成镜像。


>    build


指定 Dockerfile 所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。 Compose 将会利用它自动构建这个镜像，然后使用这个镜像。

	build: /path/to/build/dir

>    cap_add, cap_drop

指定容器的内核能力（capacity）分配。

例如，让容器拥有所有能力可以指定为：

```
cap_add:
  - ALL
```

去掉 NET_ADMIN 能力可以指定为：

```
cap_drop:
  - NET_ADMIN
```

>    command

覆盖容器启动后默认执行的命令。

```
command: echo "hello world"
```

>    cgroup_parent

指定父 cgroup 组，意味着将继承该组的资源限制。

例如，创建了一个 cgroup 组名称为 cgroups_1。

```
cgroup_parent: cgroups_1
```

>    container_name

指定容器名称。默认将会使用 项目名称_服务名称_序号 这样的格式。

```
container_name: docker-web-container
```

需要注意，指定容器名称后，该服务将无法进行扩展（scale），因为 Docker 不允许多个容器具有相同的名称。

>    devices

指定设备映射关系。

```
devices:
  - "/dev/ttyUSB1:/dev/ttyUSB0"
```

>    dns


自定义 DNS 服务器。可以是一个值，也可以是一个列表。

```
dns: 8.8.8.8
dns:
  - 8.8.8.8
  - 9.9.9.9
```

>    dns_search

配置 DNS 搜索域。可以是一个值，也可以是一个列表。

```
dns_search: example.com
dns_search:
  - domain1.example.com
  - domain2.example.com
```
>    dockerfile

如果需要指定额外的编译镜像的 Dockefile 文件，可以通过该指令来指定。

```
dockerfile: Dockerfile-alternate
```

注意，该指令不能跟 image 同时使用，否则 Compose 将不知道根据哪个指令来生成最终的服务镜像。

>    env_file

从文件中获取环境变量，可以为单独的文件路径或列表。

如果通过 docker-compose -f FILE 方式来指定 Compose 模板文件，则 env_file 中变量的路径会基于模板文件路径。

如果有变量名称与 environment 指令冲突，则按照惯例，以后者为准。

```
env_file: .env

env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

环境变量文件中每一行必须符合格式，支持 # 开头的注释行。

```
# common.env: Set development environment
PROG_ENV=development
```

>    environment




















具体的见官方文档


#	参考文档

官方文档：

[https://docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/)

第三方文档：

[https://yeasy.gitbooks.io/docker_practice/content/compose/yaml_file.html](https://yeasy.gitbooks.io/docker_practice/content/compose/yaml_file.html)

