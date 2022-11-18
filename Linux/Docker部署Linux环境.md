# 0.安装Docker

首先卸载旧版本Docker(如果未安装过可跳过)：

```bash
yum remove docker \
           docker-client \
           docker-client-latest \
           docker-common \
           docker-latest \
           docker-latest-logrotate \
           docker-logrotate \
           docker-engine
```

安装gcc环境以及所需的软件包：

```bash
yum -y install gcc
yum -y install gcc-c++
yum install -y yum-utils
```

设置stable镜像仓库以及更新yum软件包索引：

```bash
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast
```

安装DockerCE：

```bash
yum install docker-ce docker-ce-cli containerd.io
```

Docker设置开机自启状态：

```bash
# 开启自启
systemctl enable docker
# 关闭开机自启
systemctl disable docker
```

> Docker安装官方文档
>
> https://docs.docker.com/engine/install/centos/

参考文档：[Docker](Docker2022.html)

# 1.MySQL

MySQL容器运行命令：

```dockerfile
docker run -d -p 3306:3306 --privileged=true \
-v /wuse/mysql/log:/var/log/mysql \
-v /wuse/mysql/data:/var/lib/mysql \
-v /wuse/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql mysql:[tag]
```

宿主机/wuse/mysql/conf目录下新建文件my.cnf

```
# 设置字符集解决中文乱码
[client]
default_character_set=utf8
[mysqld]
collation_server = utf8_general_ci
character_set_server = utf8
```

重启MySQL容器 `docker restart mysql` 

自动启动MySQL容器 `docker update [容器id或容器名] --restart=always` 

## 1.1主从配置

首先启动两台MySQL容器，注意挂载目录与端口

```dockerfile
# 启动主数据库
docker run -d -p 3306:3306 --privileged=true \
-v /mydata/mysql-master/log:/var/log/mysql \
-v /mydata/mysql-master/data:/var/lib/mysql \
-v /mydata/mysql-master/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql-master mysql:[tag]

# 启动从数据库
docker run -d -p 3308:3306 --privileged=true \
-v /mydata/mysql-slave/log:/var/log/mysql \
-v /mydata/mysql-slave/data:/var/lib/mysql \
-v /mydata/mysql-slave/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql-slave mysql:[tag]
```

在宿主机中编辑**主数据库**配置文件 `vim /mydata/mysql-master/conf/my.cnf` (记得重启)

```
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=101 
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql  
## 开启二进制日志功能
log-bin=mall-mysql-bin  
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M  
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed  
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7  
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
```

进入**主数据库**容器并进入MySQL控制台 `docker exec -it mysql-master /bin/bash` 

```mysql
# 给主数据库创建数据同步用户
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
```

在宿主机中编辑**从数据库**配置文件 `vim /mydata/mysql-slave/conf/my.cnf` (记得重启)

```
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=102
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql  
## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用
log-bin=mall-mysql-slave1-bin  
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M  
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed  
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7  
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062  
## relay_log配置中继日志
relay_log=mall-mysql-relay-bin  
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1  
## slave设置为只读（具有super权限的用户除外）
read_only=1
```

在**主数据库**中查看主从同步状态 `show master status;` 记录下图中信息，之后要用到

![image-20220917104935621](https://gitee.com/walls1717/images/raw/master/202209171050677.png)

进入**从数据库**容器并进入MySQL控制台 `docker exec -it mysql-slave /bin/bash` 

```mysql
# 配置主从复制
change master to master_host='宿主机ip', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000001', master_log_pos=617, master_connect_retry=30;

# 参数说明
master_host：主数据库的IP地址；
master_port：主数据库的运行端口；
master_user：在主数据库创建的用于同步数据的用户账号；
master_password：在主数据库创建的用于同步数据的用户密码；
master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；
master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；
master_connect_retry：连接失败重试的时间间隔，单位为秒。
```

在**从数据库**中查看主从同步状态 `show slave status \G;` 可以看看配置的是否与刚刚输入的命令一致，主要看图中红框中的配置，No代表没有开启主从同步

![image-20220917110633802](https://gitee.com/walls1717/images/raw/master/202209171106809.png)

输入命令 `start slave;` 再次查看主从同步状态，可以看到主从同步已开启

![image-20220917110836045](https://gitee.com/walls1717/images/raw/master/202209171108055.png)



# 2.Redis

事先拷贝Redis配置文件[redis.conf](redis.conf)到宿主机目录下

在宿主机设置redis.conf

```
# 设置Redis密码(可选)
requirepass 123

# 允许redis外地连接(必须)
注释掉 # bind 127.0.0.1

# 将daemonize yes注释起来或者 daemonize no设置，因为该配置和docker run中-d参数冲突，会导致容器一直启动失败
daemonize no

# 开启redis数据持久化(可选)
appendonly yes
```

Redis容器运行命令：

```dockerfile
docker run -d -p 6379:6379 --privileged=true \
-v /mydata/redis/redis.conf:/etc/redis/redis.conf \
-v /mydata/redis/data:/data \
--name redis redis redis-server /etc/redis/redis.conf
```

自动启动Redis容器 `docker update [容器id或容器名] --restart=always` 





## 2.1主从集群配置

详情见目录下 [Docker配置Redis主从集群](Docker配置Redis主从集群.md) 



# 3.Tomcat

Tomcat容器运行命令：

```dockerfile
docker run -d -p 8080:8080 --name tomcat tomcat
```

最新版本的Tomcat在访问首页时会出现404错误，解决办法如下：

```
方法一：
# 进入Tomcat容器中tomcat的目录
/usr/local/tomcat
# 删掉webapps
rm -r webapps
# 将webapps.dist改名为webapps
mv webapps.dist webapps

方法二：
使用billygoo/tomcat8-jdk8镜像构建容器
docker run -d -p 8080:8080 --name tomcat8 billygoo/tomcat8-jdk8
```

将虚拟机中的文件复制到容器的webapps中：

 `docker cp 虚拟机中文件 容器id:/usr/local/tomcat/webapps` 

自动启动Tomcat容器 `docker update [容器id或容器名] --restart=always` 

# 4.ElaticSearch

容器运行前置操作：

```bash
# es的配置文件挂载路径
mkdir -p /mydata/elasticsearch/config
# es的数据挂载路径
mkdir -p /mydata/elasticsearch/data
# 允许所有地址访问
echo "http.host: 0.0.0.0" >> /mydata/elasticsearch/config/elasticsearch.yml
# 给读写权限
chmod -R 777 /mydata/elasticsearch/
```

ElaticSearch容器运行命令：

```dockerfile
# 9200端口用于使用es；9300端口用于kibana使用
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.4.2
```

特别注意：-e ES_JAVA_OPTS="-Xms64m -Xmx256m" \ 测试环境下，设置 ES 的初始内存和最大内存，否则导致过大启动不了ES。

自动启动ES容器 `docker update [容器id或容器名] --restart=always` 

## 1.1 Kibana

Kibana是es的可视化操作程序。

Kibana容器运行命令：

```dockerfile
docker run --name kibana \
-e ELASTICSEARCH_HOSTS=http://[虚拟机ip]:9200 -p 5601:5601 \
-d kibana:7.4.2
```

自动启动Kibana容器 `docker update [容器id或容器名] --restart=always` 

# 5.RabbitMQ

RabbitMQ容器运行命令：

```bash
docker run -d --name rabbitmq \
-p 5671:5671 -p 5672:5672 \
-p 4369:4369 -p 15671:15671 \
-p 15672:15672 rabbitmq:management

# 端口说明
4369,25672(Erlan发现&集群端口)
5672,5671(AMQP端口)
15672(web管理后台端口)
61613,61674(STOMP协议端口)
1883,8883(MQTT协议端口)
```

