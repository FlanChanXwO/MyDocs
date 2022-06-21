# Idea集成阿里云控制台
- 集成插件下载，插件名如图，在Idea插件市场搜索插件名并下载即可
 ![img.png](../../../../../../MyDocs/img/img_18.png)
-----------------------
- 步骤1：点击Add Host

![img.png](../../../../../../MyDocs/img/img_6.png)

- 步骤2：按照如图步骤进行配置，在主机列表中增加Ip地址: "106.14.206.16"
  填入你的用户名和密码，配置名称随意

![img_1.png](../../../../../../MyDocs/img/img_8.png)

- 步骤3：在配置完成后，点击 "测试连接状况" ， 查看是否能够成功连接服务器

![img_2.png](../../../../../../MyDocs/img/img_7.png)

- 步骤4：出现连接成功的提示后，即可连接服务器

![img_3.png](../../../../../../MyDocs/img/img_3.png)

![img_4.png](../../../../../../MyDocs/img/img_4.png)

成功使用终端连接到了服务器
-----------------------
  
![img_5.png](../../../../../../MyDocs/img/img_5.png)

连接服务器篇完结!
-----------------------

# 服务器配置：
### 一、安装Docker

```shell
# 1.运行以下命令，添加docker-ce的dnf源。
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


### 二、Docker中的nginx服务器的拉取与安装，启动
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


### 二、Docker中的mysql数据库服务的拉取与安装，启动，以及远程连接数据库
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
- ![img.png](../../../../../../MyDocs/img/img_13.png)
- ![img_2.png](../../../../../../MyDocs/img/img_11.png)
-------------------------
点击 "Make Global" 选项这样即可在切换不同项目时候，保持管理工具中的数据源存在
- ![img_1.png](../../../../../../MyDocs/img/img_12.png)
