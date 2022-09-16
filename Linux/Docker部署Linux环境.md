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



# 2.Redis

事先拷贝Redis配置文件redis.conf[^1]到宿主机目录下

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
-v /app/redis/redis.conf:/etc/redis/redis.conf \
-v /app/redis/data:/data \
--name redis redis redis-server /etc/redis/redis.conf
```



[^1]:详情查看目录redis.conf文件



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

