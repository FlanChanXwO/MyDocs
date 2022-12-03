# 服务器应用配置：
###  目录 >>>
#### [0.linux基本配置](#linux)
#### [1.安装Docker](#docker)
#### [2.Docker中的nginx服务器的拉取与安装，启动](#nginx)
#### [3.Docker中的mysql数据库服务的拉取与安装，启动，以及远程连接数据库](#mysql)
#### [4.1.ElasticSearch的部署](#es)
#### [4.2.Kibana的部署](#kibana)
#### [5.RabbitMQ消息队列的部署](#rabbittmq)
#### [6.Redis分布式缓存的部署](#redis)
#### [7.Nacos注册中心的部署](#nacos)
#### [8.GateWay网关的部署](#gateway)
#### [9.hadoop的部署](#hadoop)
#### [10.kafaka的部署](#kafaka)
#### [11.hadoop完全分布式的部署](#hadoop_distribution)


## <span id="linux">0. Linux基本配置</span>
### 卸载Linux的图形化界面
> 在命令符界面输入以下命令，即可完成图形化界面的卸载
```shell
# 查看默认的target，执行：
systemctl get-default
# 开机以命令模式启动，执行：
systemctl set-default multi-user.target
# 开机以图形界面启动，执行：
systemctl set-default graphical.target
```
> 修改主机名
```shell
vim /etc/hostname
```
> 修改主机映射
```shell
vim /etc/hosts
```
> 修改Ip地址和网关信息
```shell
vim /etc/sysconfig/network-scripts/ifcfg-ens33
```
> 例： <br/>
> IPADDR=192.168.10.102 <br/>
> PREFIX=24 <br/>
> GATEWAY=192.168.10.2 <br/>
> DNS1=192.168.10.2 <br/>

## <span id="docker">一、安装Docker</span>
```shell
# 1.运行以下命令，安装cnf软件包管理器，添加docker-ce的dnf源。
sudo yum -y install dnf
sudo dnf config-manager --add-repo=https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 2.运行以下命令，安装docker-ce。
sudo dnf -y install docker-ce
# 3.运行以下命令，查看docker-ce是否成功安装
sudo dnf list docker-ce
# 4.运行以下命令，启动Docker服务
sudo systemctl start docker
# 5.运行以下命令，查看Docker服务的运行状态 然后不断按Enter键，直到安装完成
sudo systemctl status docker
# 6.安装完后 成功启动docker 查看docker版本，验证是否验证成功
sudo docker -v
# 7.用户组添加，赋予普通用户使用docker的权限
sudo groupadd docker 
sudo gpasswd -a ${USER} docker
# 更新用户组
newgrp docker
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
```nginx configuration
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

	# 反向代理配置
	upstream server_list{
	   # 负载均衡算法
	   # ip_hash; 按照IP地址的哈希值分配 
	   # least_conn; 最少连接
	   # server localhost:8900 weight=5; 轮询权重分配
	   server localhost:8080;
	   server localhost:9999;
	}
	
	
    
    server {
        listen 80; #监听80端口号
        server_name 106,14,296.16; #主机名。
        location / {
            root html;
            index index.html index.htm;
        }
        location /proxy {
            # 反向代理
            proxy_pass http://server_list;
        }
         #拦截静态资源
#         location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|js|css)$ {
#         root static;
#         expires      30d;  
#        }
    }
    
    server {
        listen 443;
        server_name 106,14,296.16;
        location / {
            # 重定向
            rewrite 106,14,296.16:80;
        }
    }

    include /etc/nginx/conf.d/*.conf;
}
```
### 设置nginx配置文件 (SSL证书)
```nginx configuration
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
            listen 443 ssl;
            #对应你的域名
            server_name flanzone.com;
            ssl_certificate ./cert/ssl.crt;
            ssl_certificate_key ./ssl.key;
            ssl_session_timeout 5m;
            ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
            ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
            ssl_prefer_server_ciphers on;
            #如果是静态文件,直接指向目录,如果是动态应用,用proxy_pass转发一下
            location / {
                     root html;
                     index index.html index.htm;
            }
        }
        
        #监听80端口,并重定向到443
        server {
            listen 80;
            server_name 43.142.149.6;
            rewrite ^/(.*)$ https://www.flanzone.com:443/$1 permanent;
        }
    
        include /etc/nginx/conf.d/*.conf;
}
```

- 步骤3：在~/nginx目录下，进行docker容器的拉取操作并安装nginx到容器中
```shell
docker pull nginx:1.22.0
cd ~/nginx
docker run -id --name=nacos_nginx \
-p 8440:8440 \
-v $PWD/conf/cert:/etc/nginx/cert \
-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
-v $PWD/logs:/var/log/nginx \
-v $PWD/html:/etc/nginx/html/ \
--restart=always \
nginx:1.22.0
```

## <span id="mysql">二、Docker中的mysql数据库服务的拉取与安装，启动，以及远程连接数据库</span>
- 步骤1：mysql数据库的拉取与安装，启动
##### 执行下列代码，完成步骤1的所需操作
```shell
docker pull mysql:5.7.30
mkdir ~/mysql
```
#### 实例化并运行mysql数据库
```shell
cd ~/mysql
docker run -id  --privileged=true --name=mysql_c1 -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=Dlh123456789 -p 3306:3306 mysql:5.7.30 
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
    --restart=always \
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


## <span id="rabbittmq">五、RabbitMQ消息队列部署</span>

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
redis镜像拉取,版本6.x
```shell
docker pull redis:6.2.7
```
执行下面的shell语句来部署单点redis。
```sh
docker run -id --name=redis_single -p 6379:6379 -v $PWD/conf/redis.conf:/etc/redis/redis.conf -v $PWD/data:/data --restart=always -d redis:6.2.7 redis-server /etc/redis/redis.conf 
``` 

## <span id="nacos">七、Nacos注册中心的应用部署</span>


# 创建自定义网络
```sh
docker network create --driver bridge --subnet 47.108.78.230/22 nacos_network
```

# 查看已存在网络
```sh
docker network ls
```

# 执行下面的shell语句来部署nacos

```sh
docker pull nacos/nacos-server:1.3.1
docker run -d -p 8848:8848 \
--name nacos_single \
--network nacos_network \
-e prefer_host_mode=192.168.10.100 \
--env MODE=standalone \
--env SPRING_DATASOURCE_PLATFORM=mysql \
--env MYSQL_SERVICE_HOST=192.168.10.100 \
--env MYSQL_SERVICE_PORT=3306 \
--env MYSQL_SERVICE_DB_NAME=nacos_config \
--env MYSQL_SERVICE_USER=root \
--env MYSQL_SERVICE_PASSWORD=mysql \
--restart=always \
-v $PWD/logs:/home/nacos/logs \
nacos/nacos-server:1.3.1
```



## <span id="gateway">八、gateway的应用部署</span>
首先将打包好的容器进行解压
```sh
docker build -t flanchan/gateway:1.0 .
``` 
执行下面的shell语句来部署gateway。
```sh
docker run --name MyGateway -p 10010:10010 -d flanchan/gateway:1.0
``` 



## <span id="hadoop">九、hadoop的应用部署</span>
首先拉取hadoop镜像
```sh
docker pull sequenceiq/hadoop-docker:2.7.1
```
拉取完镜像后，启动容器
```sh
docker run -dit --name hadoop_single --privileged=true -p 9870:9870 -p 19888:19888 -p 50070:50070 -p 8088:8088  -p 9000:9000 sequenceiq/hadoop-docker:2.7.1 /etc/bootstrap.sh -bash
```
启动容器成功后，进入容器并添加环境变量信息
```sh
docker exec -it hadoop_single /bin/bash
PATH=$PATH:/usr/local/hadoop/bin/
```
然后就可以通过web来访问hadoop了
> 查看集群状态：IP:8088 <br/>
> 浏览HDFS文件：IP:50070



## <span id="kafaka">十、kafaka的应用部署</span>
拉取镜像：由于kafka需要zookeeper管理,因此需要安装zookeeper的镜像
```sh
docker pull wurstmeister/zookeeper
docker pull wurstmeister/kafka
```
启动kafka需要基于zookeeper管理，因此需要启动2个容器
```sh
docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper
docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=192.168.118.132:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.118.132:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka
```
> 注意："KAFKA_ZOOKEEPER_CONNECT"和"KAFKA_ADVERTISED_LISTENERS"这两个参数需要换成你自己的宿主机ip,否则启动会失败,
> 导致在别的机器上访问不到kafka
# 测试消息订阅发送
进入kafka命令执行页面
```sh
docker exec -it kafka bash
cd ./opt/kafka_2.13-2.8.1/
```
在kafka命令执行页面创建topic,生产消息
输入以下命令后，可进行消息输入，输入任意消息
```sh
./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mykafka
```
订阅topic消息，获取之前生产的消息
```sh
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mykafka --from-beginning
```
如果能获取到之前生产的消息，就说明测试成功


## <span id="hadoop_distribution">十一、hadoop完全分布式的应用部署</span>
拉取基础镜像
```shell
docker pull centos:centos7
```
检查镜像是否拉取成功
```shell
docker images
```
启动基础镜像
```shell
docker run -d -id --name base centos:centos7 /bin/bash
```
复制软件到基础镜像中
```shell
docker cp ./java-1.8 base:/opt/ && docker cp ./hadoop-3.1.3 base:/opt/
```
进入容器
```shell
docker exec -it base bash
```
为基础镜像安装软件
```shell
yum -y install vim && yum -y install net-tools && yum -y install openssh-clients && yum -y install openssh-server && yum -y install openssl && yum -y install wget
```
关闭防火墙
```shell
systemctl stop firewalld
systemctl disable firewalld.service
```
修改root用户密码
```shell
passwd flanchan
```
打包已安装好软件的基础镜像
```shell
docker commit base flanchan/centos7:latest
```
### 完全分布式部署
```shell
cd /opt
```
设置环境变量
```shell
vim /etc/profile.d/my_env.sh
```
```shell
#JAVA_HOME
export JAVA_HOME=/opt/java-1.8
export PATH=$PATH:$JAVA_HOME/bin
#HADOOP_HOME
export HADOOP_HOME=/opt/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
#HDFS_USER
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
#YARN_USER
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```
```shell
# 环境变量载入
source /etc/profile
```
镜像提交
```shell
docker commit base flanchan/hadoop:3.1.3
```
创建网络
```shell
docker network create hadoop_network
```
编写docker-compose.yml文件
```yaml
version: "3.3"
services:
  namenode:
    ports:
      - "9870:9870"
      - "9000:9000"
    image: "flanchan/hadoop:3.1.3"
    networks:
      - hadoop_network
    container_name: "namenode"
    tty: true
    restart: always
    privileged: true
    command: /usr/sbin/init
  secondarynamenode:
    ports:
      - "9868:9868"
    image: "flanchan/hadoop:3.1.3"
    networks:
      - hadoop_network
    container_name: "secondarynamenode"
    tty: true
    restart: always
    privileged: true
    command: /usr/sbin/init
  resourcemanager:
    ports:
      - "8032:8032"
    image: "flanchan/hadoop:3.1.3"
    networks:
      - hadoop_network
    container_name: "resourcemanager"
    tty: true
    restart: always
    privileged: true
    command: /usr/sbin/init
  jobhistory:
    ports:
      - "19888:19888"
    image: "flanchan/hadoop:3.1.3"
    networks:
      - hadoop_network
    container_name: "jobhistory"
    tty: true
    restart: always
    privileged: true
    command: /usr/sbin/init
  slave1:
    image: "flanchan/hadoop:3.1.3"
    networks:
      - hadoop_network
    container_name: "slave1"
    tty: true
    restart: always
    privileged: true
    command: /usr/sbin/init
  slave2:
    image: "flanchan/hadoop:3.1.3"
    networks:
      - hadoop_network
    container_name: "slave2"
    tty: true
    restart: always
    privileged: true
    command: /usr/sbin/init
networks:
  hadoop_network:
    external: true
```
下载docker-compose
```shell
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# 赋予用户使用权限
sudo chmod +x /usr/local/bin/docker-compose
```
启动集群
```shell
docker compose up -d
```
配置免密登录
```shell
docker exec -it namenode bash
```
为每个机子设置新的认证密码
```shell
passwd root
flanchan
flanchan
```
编辑主机映射文件
```shell
vim /etc/hosts
```
在hosts文件中添加
```shell
172.18.0.2 slave2
172.18.0.3 slave1
172.18.0.4 secondarynamenode
172.18.0.5 client
172.18.0.6 namenode
172.18.0.7 jobhistory
172.18.0.8 resourcemanager
```
编辑主机名称 (可选)
```shell
vim /etc/hostname #根据应用名命名即可
```
sudo /etc/init.d/ssh start
```shell
mkdir /root/.ssh
cd /root/.ssh
ssh-keygen -t rsa
# 然后敲三个回车，会生成id_rsa(私钥)，id_rsa.pub(公钥)
```


将公钥复制到要免密登录的机器上
```shell
ssh-copy-id namenode
ssh-copy-id secondarynamenode
ssh-copy-id resourcemanager
ssh-copy-id jobhistory
ssh-copy-id slave1
ssh-copy-id slave2
ssh-copy-id client
```
> 注意每个机器都要重复一次上述免密登录的步骤
进入任意容器启动集群
```shell
docker exec -it namenode bash
```
```shell
hdfs namenode -format
start-dfs.sh
start-yarn.sh
mr-jobhistory-daemon.sh start historyserver
```


docker run --hostname=hadoop_single \ -p 8088:8088 -p 9870:9870 -p 9864:9864 \ -p 19888:19888 -p 8042:8042 -p 8888:8888 \ --name hadoop_single -d hadoop:3