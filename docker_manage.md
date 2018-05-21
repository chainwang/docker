#	简介

为了提高服务器的利用率，我们可以使用docker技术。

#	安装

现在centos已经引进了docker技术，我们可以通过yum安装。

>	安装yum扩展

	yum install -y yum-utils
    
>	配置yum源
    
	yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

	yum-config-manager --enable docker-ce-edge
    
>	添加缓存

	yum makecache fast

>  更新
	
    yum update

>	安装docker-ce版本
	
    yum install docker-ce
    
>	查看docker信息
   
	yum list docker-ce.x86_64  --showduplicates |sort -r
    
>	修改docker镜像存储目录

docker镜像默认目录在/var/lib/docker/

docker镜像存储目录是改不了的，如果机器的镜像不多，可以不用修改。

修改有两种方式

一种是给/var/lib/docker/目录挂载一个新的磁盘

另一种是为该目录添加一个软连接

	mv  /var/lib/docker/  /data/docker/root_dir/
    
    ln -s /data/docker/root_dir/  /var/lib/docker/
    
>	启动
	
    systemctl start docker

>	加入开启启动

	systemctl  enable docker   
    

    
#	使用


##	使用镜像

>	查看docker版本

	docker version
    
>	查看详细信息

	docker info

>	查询docker镜像

查询centos镜像

	docker search centos


>	下载镜像

下载centos镜像，默认下载最新版本，如果需要下载其他版本，例如“docker pull centos:centos6”

	docker pull centos
    
>	启动一个容器

启动centos并重新命名为test1

	docker run --name test1 -i -t centos /bin/bash
    
docker run  = docker create +docker start


>	查看所有docker运行状态

	docker ps -a

>	停止一个docker容器

	docker rm -f "CONTAINED ID"
    
>	删除docker镜像
	
	docker rmi “镜像名字”




>	运行一个带数据访问端口的容器

	docker run -p 4000:80 nginx
    
“-p”:指定容器到系统的端口映射，前面是服务器端口，后面是容器端口。

>	后台运行

	docker run -d -p 4000:80 nginx
    
"-d"：后台运行

>	进入容器

	docker exec -it test1 /bin/bash
    
>	文件挂载
  
	docker run --name  tomcat  -d -p 8082:8080 -v /root/test:/root/test tomcat

“-v”：系统文件挂载到容器内部，实现容器和系统的文件共享，前面是服务器文件位置，后面是容器文件位置。

##	构建镜像

>	重新保存已经修改的镜像

由于镜像不能修改，只能修改容器内的数据，容器内的数据不能持久化保存，

我们进入容器修改后，如果很重要就要重新保存数据。

	docker commit  --author  "cheng"  --message "change index.html" webserver nginx:v2

其中 --author 是指定修改的作者，而 --message 则是记录本次修改的内容，webserver 是正在运行容器的名字。

##	镜像迁移

>	镜像打包

	docker save -o ubuntu.tar.gz   ubuntu
    
>	打包镜像导入

	docker load --input ubuntu.tar.gz  
    
##	制定镜像

使用 Dockerfile 定制镜像，分2步

*	编写Dockerfile

*	docker build -t "镜像名":"版本"  .



###	简单定制一个镜像

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。最多不能超出127层。

为了减少层数，我们需要这样写

```
ROM debian:jessie

RUN buildDeps='gcc libc6-dev make' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```


docker构建一个新的镜像需要一个Dockerfile,里面是写的脚本。

简单的制定一个镜像

	mkdir  mynginx
    cd  mynginx
    vim Dockerfile

写入下面内容

```    
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html

```

	docker build -t nginx:v3 .

会显示以下这些内容，表示你已经安装成功

```
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM nginx
 ---> 2f7f7bce8929
Step 2/2 : RUN echo '<h1>Hello,My name is Casent</h1>' > /usr/share/nginx/html/index.html
 ---> Running in 67e5c9fd1068
 ---> 797a13583afb
Removing intermediate container 67e5c9fd1068
Successfully built 797a13583afb
Successfully tagged nginx:v3


```

###	Dockerfile指令详解

####	镜像构建上下文（Context）

docker build 命令最后有一个“ .”。“. ”表示当前目录，而 Dockerfile 就在当前目录，因此不少初学者以为这个路径是在指定 Dockerfile 所在路径，这么理解其实是不准确的。如果对应上面的命令格式，你可能会发现，这是在指定上下文路径。


####	FROM

FROM 指定基础镜像

所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个 nginx 镜像的容器，再进行修改一样，基础镜像是必须指定的。而 FROM 就是指定基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。

除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 scratch。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。

```
FROM scratch
...

```
如果你以 scratch 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

不以任何系统为基础，直接将可执行文件复制进镜像的做法并不罕见，比如 swarm、coreos/etcd。对于 Linux 下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接 FROM scratch 会让镜像体积更加小巧。


####	RUN

RUN 执行命令

RUN 指令是用来执行命令行命令的。由于命令行的强大能力，RUN 指令在定制镜像时是最常用的指令之一。其格式有两种：

*	shell 格式：RUN <命令>，就像直接在命令行中输入的命令一样。刚才写的 Dockrfile 中的 RUN 指令就是这种格式。

```
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```


*	exec 格式：RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。

```
FROM debian:jessie

RUN apt-get update
RUN apt-get install -y gcc libc6-dev make
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz"
...

```

####	COPY

COPY 复制文件


COPY 指令将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置

将本地的“package.json”，放到容器“/usr/src/app/”中，可以使用通配符。

	COPY package.json /usr/src/app/


####	ADD

ADD 更高级的复制文件，ADD 指令和 COPY 的格式和性质基本一致。

如果 <源路径> 为一个 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，ADD 指令将会自动解压缩这个压缩文件到 <目标路径> 去。

直接解压一个系统作为一个镜像

```
FROM scratch
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /

```


####	CMD

CMD 容器启动命令，CMD 指令的格式和 RUN 相似。

*	shell 格式：CMD <命令>
*	exec 格式：CMD ["可执行文件", "参数1", "参数2"...]

在指令格式上，一般推荐使用 exec 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 "，而不要使用单引号。

提到 CMD 就不得不提容器中应用在前台执行和后台执行的问题。

Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 upstart/systemd 去启动后台服务，容器内没有后台服务的概念。

如果将 CMD 写为：

	CMD service nginx start
    
然后发现容器执行后就立即退出了。甚至在容器内去使用 systemctl 命令结果却发现根本执行不了。这就是因为没有搞明白前台、后台的概念，没有区分容器和虚拟机的差异，依旧在以传统虚拟机的角度去理解容器。

对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

而使用 service nginx start 命令，则是希望 upstart 来以后台守护进程形式启动 nginx 服务。而刚才说了 CMD service nginx start 会被理解为 CMD [ "sh", "-c", "service nginx start"]，因此主进程实际上是 sh。那么当 service nginx start 命令结束后，sh 也就结束了，sh 作为主进程退出了，自然就会令容器退出。

正确的做法是直接执行 nginx 可执行文件，并且要求以前台形式运行。比如：

	CMD ["nginx", "-g", "daemon off;"]
    

####    ENTRYPOINT

ENTRYPOINT 入口点

ENTRYPOINT 的格式和 RUN 指令格式一样，分为 exec 格式和 shell 格式。

ENTRYPOINT就是可以让docker运行的时候可以传递参数到容器内部。

构建容器是加入

    ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]
    
运行docker时，就可以加入参数了
    
    docker run myip -i

####    ENV

置环境变量

格式有两种：

*   ENV <key> <value>

*   ENV <key1>=<value1> <key2>=<value2>...

``` 

ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"

```

####    ARG

格式：

ARG <参数名>[=<默认值>]

ARG 构建参数和 ENV 的效果一样，都是设置环境变量。所不同的是，ARG 所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用 ARG 保存密码之类的信息，因为 docker history 还是可以看到所有值的。



####    VOLUME

VOLUME 定义匿名卷

格式为：

*   VOLUME ["<路径1>", "<路径2>"...]
*   VOLUME <路径>
   

    VOLUME /data
    
    
这里的 /data 目录就会在运行时自动挂载为匿名卷，任何向 /data 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化.


    docker run -d -v mydata:/data nginx
    
在这行命令中，就使用了 mydata 这个命名卷挂载到了 /data 这个位置，替代了 Dockerfile 中定义的匿名卷的挂载配置。


####    EXPOSE

EXPOSE 声明端口

格式为：

EXPOSE <端口1> [<端口2>...]

EXPOSE 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。
    
EXPOSE 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。


####    WORKDIR

格式为：

WORKDIR <工作目录路径>。

使用 WORKDIR 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，WORKDIR 会帮你建立目录。


如果你这样写

```
RUN cd /app
RUN echo "hello" > world.txt

```

这个 Dockerfile 进行构建镜像运行后，会发现找不到 /app/world.txt 文件，或者其内容不是 hello。原因是：在Shell中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令；而在 Dockerfile 中，这两行 RUN 命令的执行环境根本不同，是两个完全不同的容器。这就是对 Dokerfile 构建分层存储的概念不了解所导致的错误。


每一个 RUN 都是启动一个容器、执行命令、然后提交存储层文件变更。第一层 RUN cd /app 的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。

因此如果需要改变以后各层的工作目录的位置，那么应该使用 WORKDIR 指令。





























具体的见官方文档


#	参考文档

官方文档：

[https://docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/)

第三方文档：

[https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/cmd.html](https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/cmd.html)

