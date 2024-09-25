# Docker

#### <span id="catalogue">目录</span>

#### [Docker基本操作](#basic)

#### [数据卷](#volume)

#### [自定义镜像](#custom)

#### [网络](#network)

### <span id="basic">[Docker基本操作](#catalogue)</span>

<div>
<span>
当我们利用Docker安装应用时，Docker会自动搜索并下载应用镜像（image）。镜像不仅包含应用本身，还包含应用运行所需要的环境、配置、系统函数库。Docker会在运行镜像时创建一个隔离环境，称为容器（container）。
</span>
<span><br/>
镜像仓库：存储和管理镜像的平台，Docker官方维护了一个公共仓库：Docker Hub。
</span>
</div>
<br/>

#### [文档地址](https://docs.docker.com/)

拉取镜像

```shell
docker pull
```

推送镜像

```shell
docker push
```

删除镜像

```shell
docker rmi
```

删除容器

```shell
docker rm #删除一个未运行中的容器
docker rm -f #强制删除一个容器
```

列出容器信息

```shell
docker ps #列出所有运行中的容器
docker ps -a #列出所有容器
```

列出镜像

```shell
docker images
```

查看容器日志

```shell
docker logs
```

对容器执行指定命令

```shell
docker exec
```

载入镜像

```shell
docker load
```

导出镜像

```shell
docker save
```

构建镜像

```shell
docker build
```

从配置到运行容器

```shell
docker run
```

重启容器

```shell
docker restart
```

停止容器

```shell
docker stop
```

运行容器

```shell
docker start
```

### <span id="volume">[数据卷](#catalogue)</span>

<div>
数据卷是一个虚拟目录，它将宿主机目录映射到容器内目录，方便我们操作容器内文件，或者方便迁移容器产生的数据
</div><br/>

[创建数据卷](https://docs.docker.com/reference/cli/docker/volume/create/)

```shell
docker volume create
```

[查看所有数据卷](https://docs.docker.com/engine/reference/commandline/volume_ls/)

```shell
docker volume ls
```

[删除指定数据卷](https://docs.docker.com/engine/reference/commandline/volume_prune/)

```shell
docker volume rm
```

[查看某个数据卷的详情](https://docs.docker.com/engine/reference/commandline/volume_inspect/)

```shell
docker volume inspect
```

[清除数据卷](https://docs.docker.com/engine/reference/commandline/volume_prune/)

```shell
docker volume prune
```

### <span id="custom">[自定义镜像](#catalogue)</span>

![docker_custom_1.png](../MyDocs//img/docker_custom_1.png)

<div>
镜像就是包含了应用程序、程序运行的系统函数库、运行配置等文件的文件包。构建镜像的过程其实就是把上述文件打包的过程。
</div>
<br/>

![docker_custom_2.png](../MyDocs/img/docker_custom_2.png)

#### DockerFile

Dockerfile就是一个文本文件，其中包含一个个的指令(Instruction)，用指令来说明要执行什么操作来构建镜像。将来Docker可以根据Dockerfile帮我们构建镜像。常见指令如下

![docker_custom_3.png](../MyDocs/img/docker_custom_3.png)

基于centos基础镜像，利用Dockerfile描述镜像结构

```dockerfile
# 指定基础镜像
FROM centos:7
# 配置环境变量，JDK的安装目录、容器内时区
ENV JAVA_DIR=/usr/local
# 拷贝jdk和java项目的包
COPY ./jdk8.tar.gz $JAVA_DIR/
COPY ./docker-demo.jar /tmp/app.jar
# 安装JDK
RUN cd $JAVA_DIR \ && tar -xf ./jdk8.tar.gz \ && mv ./jdk1.8.0_144 ./java8
# 配置环境变量
ENV JAVA_HOME=$JAVA_DIR/java8
ENV PATH=$PATH:$JAVA_HOME/bin
# 入口，java项目的启动命令
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

我们可以基于Centos基础镜像，利用Dockerfile描述镜像结构
也可以直接基于JDK为基础镜像，省略前面的步骤：

```dockerfile
# 基础镜像
FROM openjdk:11.0-jre-buster
# 拷贝jar包
COPY docker-demo.jar /app.jar
# 入口
ENTRYPOINT ["java", "-jar", "/app.jar"] 
```

当编写好了Dockerfile，可以利用下面命令来构建镜像

```shell
docker build -t myImage:1.0 .
```

<span style="color:red">-t</span> ：是给镜像起名，格式依然是repository:tag的格式，不指定tag时，默认为latest
<br/>
<span style="color:red">.</span>  ：是指定Dockerfile所在目录，如果就在当前目录，则指定为"."

### <span id="network">[网络](#catalogue)</span>

![docker_network_1.png](../MyDocs/img/docker_network_1.png)

[创建一个网络](https://docs.docker.com/engine/reference/commandline/network_create/)

```shell
docker network create
```

[查看所有网络](https://docs.docker.com/engine/reference/commandline/network_ls/)

```shell
docker network ls
```

[删除指定网络](https://docs.docker.com/engine/reference/commandline/network_rm/)

```shell
docker network rm
```

[清除未使用的网络](https://docs.docker.com/engine/reference/commandline/network_prune/)

```shell
docker network prune
```

[使指定容器连接加入某网络](https://docs.docker.com/engine/reference/commandline/network_connect/)

```shell
docker network connect
```

[使指定容器连接离开某网络](https://docs.docker.com/engine/reference/commandline/network_disconnect/)

```shell
docker network disconnect
```

[查看网络详细信息](https://docs.docker.com/engine/reference/commandline/network_inspect/)

```shell
docker network inspect
```
