# 服务器应用配置：
### 跳转链接：
#### [1.安装Docker](#docker)
#### [2.Docker中的nginx服务器的拉取与安装，启动](#nginx)
#### [3.Docker中的mysql数据库服务的拉取与安装，启动，以及远程连接数据库](#mysql)
#### [4.ElasticSearch的部署](#es)
#### [5.Kibana的部署](#kibana)
#### [6.RocketMQ消息队列的部署](#rocketmq)
## <span id="docker">一、安装Docker</span>
```shell
# 1.运行以下命令，安装cnf软件包管理器，添加docker-ce的dnf源。
yum -y install dnf
dnf config-manager --add-repo=https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 2.运行以下命令，安装Alibaba Cloud Linux 3专用的dnf源兼容插件
dnf -y install dnf-plugin-releasever-adapter --repo alinux3-plus
# 3.运行以下命令，安装docker-ce。
dnf -y install docker-ce --nobest
# 4.运行以下命令，查看docker-ce是否成功安装
dnf list docker-ce
# 5.运行以下命令，启动Docker服务
systemctl start docker
# 6.运行以下命令，查看Docker服务的运行状态 然后不断按Enter键，直到安装完成
systemctl status docker
# 7.安装完后 成功启动docker 查看docker版本，验证是否验证成功
docker -v
```


### <span id="nginx">二、Docker中的nginx服务器的拉取与安装，启动</span>
- 步骤1：首先创建目录，并进行相关nginx的配置文件的相关操作
#### 实例化并运行nginx
```shell
# 在/root目录下创建nginx目录用于存储nginx数据信息
mkdir ~/nginx
cd ~/nginx
mkdir conf
cd conf
mkdir cert
# 在~/nginx/conf/下创建nginx.conf文件,粘贴下面内容
vim nginx.conf
```
- 步骤2：在nginx.conf文件中，复制粘贴以下内容。然后按下"ESC"键 + 组合键"Shift + :"，并输入"wq"，保存退出
#### 设置nginx配置信息
```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
worker_connections  1024;
}


http {
include       /etc/nginx/mime.types;
default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;
    
    server {
        listen 80; #监听80端口号
        server_name 106.14.206.16; #主机名。
        location / {
            root html;
            index index.html index.htm;
        }
    }

    include /etc/nginx/conf.d/*.conf;
}
```
- 步骤3：在~/nginx目录下，进行docker容器的拉取操作并安装nginx到容器中
```shell
docker pull nginx
cd ~/nginx
docker run -id --name=nginx_01 \
-p 80:80 \
-p 443:443 \
-v $PWD/conf/cert:/etc/nginx/cert \
-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
-v $PWD/logs:/var/log/nginx \
-v $PWD/html:/etc/nginx/html/ \
nginx
```

## <span id="mysql">二、Docker中的mysql数据库服务的拉取与安装，启动，以及远程连接数据库</span>
- 步骤1：mysql数据库的拉取与安装，启动
##### 执行下列代码，完成步骤1的所需操作
```shell
docker pull mysql:8.0.29
mkdir ~/mysql
```
#### 实例化并运行mysql数据库
```shell
cd ~/mysql
docker run -id --name=mysql_c1 -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=Dlh123456789 -p 3306:3306 mysql:8.0.29 
```
###### 参数说明
> - --name：实例名称
> - -p port:port：端口映射
> - -v 数据卷配置，实现数据持久化，以及实现与外部机器进行数据交换
> - MYSQL_ROOT_PASSWORD：数据库密码设置

- 步骤2：通过SQL图形化管理工具与mysql数据库进行远程连接
##### 首先创建新的数据库连接配置
创建新的数据库连接配置，测试连接成功后即可Apply->OK
- ![img.png](/img/img_13.png)
- ![img_2.png](/img/img_11.png)
-------------------------
点击 "Make Global" 选项这样即可在切换不同项目时候，保持管理工具中的数据源存在
- ![img_1.png](/img/img_12.png)


## <span id="es">三、ElasticSearch分布式搜索引擎部署</span>

## 3.部署单点es

## 3.1.创建网络

因为我们还需要部署kibana容器，因此需要让es和kibana容器互联。这里先创建一个网络：

```sh
docker network create es-net
```

## 3.2.加载镜像
```sh
# 以本地方式导入镜像
docker load -i es.tar
```
```sh
# 以在线方式导入镜像
docker pull elasticsearch:latest
```
## 3.3.运行

运行docker命令，部署单点es：

```sh
docker run -d \
	--name es \
    -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
    -e "discovery.type=single-node" \
    -v es-data:/usr/share/elasticsearch/data \
    -v es-plugins:/usr/share/elasticsearch/plugins \
    --privileged \
    --network es-net \
    -p 9200:9200 \
    -p 9300:9300 \
elasticsearch:7.12.1
```

命令解释：
- `-e "cluster.name=es-docker-cluster"`：设置集群名称
- `-e "http.host=0.0.0.0"`：监听的地址，可以外网访问
- `-e "ES_JAVA_OPTS=-Xms512m -Xmx512m"`：内存大小
- `-e "discovery.type=single-node"`：非集群模式
- `-v es-data:/usr/share/elasticsearch/data`：挂载逻辑卷，绑定es的数据目录
- `-v es-logs:/usr/share/elasticsearch/logs`：挂载逻辑卷，绑定es的日志目录
- `-v es-plugins:/usr/share/elasticsearch/plugins`：挂载逻辑卷，绑定es的插件目录
- `--privileged`：授予逻辑卷访问权
- `--network es-net` ：加入一个名为es-net的网络中
- `-p 9200:9200`：端口映射配置


## <span id="kibana">四、Kibana可视化数据展示平台部署</span>

## 4.1.部署

运行docker命令，部署kibana

```sh
docker run -d \
--name kibana \
-e ELASTICSEARCH_HOSTS=http://es:9200 \
--network=es-net \
-p 5601:5601  \
kibana:7.12.1
```

- `--network es-net` ：加入一个名为es-net的网络中，与elasticsearch在同一个网络中
- `-e ELASTICSEARCH_HOSTS=http://es:9200"`：设置elasticsearch的地址，因为kibana已经与elasticsearch在一个网络，因此可以用容器名直接访问elasticsearch
- `-p 5601:5601`：端口映射配置


## <span id="rocketmq">五、RocketMQ消息队列部署</span>

# 5.1单机部署

## 5.1.1下载镜像

方式一：在线拉取

``` sh
docker pull rabbitmq:3-management
```

方式二：从本地加载

```sh
docker load -i mq.tar
```

## 5.1.2安装MQ

执行下面的命令来运行MQ容器：

```sh
docker run \
 -e RABBITMQ_DEFAULT_USER=itcast \
 -e RABBITMQ_DEFAULT_PASS=123321 \
 --name mq \
 --hostname mq1 \
 -p 15672:15672 \
 -p 5672:5672 \
 -d \
 rabbitmq:3-management
```

# 5.2集群部署

接下来，我们看看如何安装RabbitMQ的集群。

## 5.2.1集群分类

在RabbitMQ的官方文档中，讲述了两种集群的配置方式：

- 普通模式：普通模式集群不进行数据同步，每个MQ都有自己的队列、数据信息（其它元数据信息如交换机等会同步）。例如我们有2个MQ：mq1，和mq2，如果你的消息在mq1，而你连接到了mq2，那么mq2会去mq1拉取消息，然后返回给你。如果mq1宕机，消息就会丢失。
- 镜像模式：与普通模式不同，队列会在各个mq的镜像节点之间同步，因此你连接到任何一个镜像节点，均可获取到消息。而且如果一个节点宕机，并不会导致数据丢失。不过，这种方式增加了数据同步的带宽消耗。

我们先来看普通模式集群。

## 5.2.2设置网络

首先，我们需要让3台MQ互相知道对方的存在。

分别在3台机器中，设置 /etc/hosts文件，添加如下内容：

```
ip1 mq1
ip2 mq2
ip3 mq3
```

- ip1 ~ ipn 为具体的IP地址，保证每个IP地址不同即可


## <span id="redis">六、Redis的应用部署</span>

执行下面的shell语句来部署redis。

```sh
docker run --name redis -d redis:6.2.7 redis-server --save 60 1 --loglevel warning
```