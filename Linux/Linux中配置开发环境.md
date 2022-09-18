# 1.关闭防火墙

```txt
关闭/开启/重启 防火墙
systemctl stop/start/restart firewalld
禁止防火墙开机自启
systemctl disable firewalld

开放指定端口
firewall-cmd --zone=public --add-port=80/tcp --permanent
关闭指定端口
firewall-cmd --zone=public --remove-port=8080/tcp --permanent
重启防火墙
firewall-cmd --reload
查看开放的端口
firewall-cmd --zone=public --list-ports
```



# 2.JDK

1. 使用ssh工具将JDK的二进制发布包上传到Linux上

2. `tar -zxvf 压缩包名 -C 被解压到的路径` 压缩包名就是你上传的二进制发布包的包名，被解压到的路径就是你想把这个发布包解压到那个地方

3. 配置环境变量，这里要用到 vim，如果Linux上没有安装过vim，使用 `yum install vim` 安装vim编辑器  `vim /etc/profile` 在文件末尾加入如下配置

   ```
   JAVA_HOME=/usr/local/jdk1.8.0_171 #这里的文件名根据自己情况而定
   PATH=$JAVA_HOME/bin:$PATH
   ```

4. `source /etc/profile` 重新加载profile文件，使配置生效

5. `java -version` 检查是否配置成功



# 3.MySQL

1. 检查Linux是否已经安装如下程序

   ```
   rpm -qa | grep mysql
   rpm -qa | grep mariadb (centOS一般自带这个)
   ```

   ![image-20220616115515947](https://gitee.com/walls1717/images/raw/master/image-20220616115515947.png)

   这个就代表系统已经安装mariadb，这个与MySQL有冲突，需要卸载

2. 卸载已安装的冲突软件

   ```
   rpm -e --nodeps 软件名称
   ```

3. 使用rpm安装MySQL

   ```
   rpm -ivh mysql-community-common-5.7.25-1.el7.x86_64.rpm
   rpm -ivh mysql-community-libs-5.7.25-1.el7.x86_64.rpm
   rpm -ivh mysql-community-devel-5.7.25-1.el7.x86_64.rpm
   rpm -ivh mysql-community-libs-compat-5.7.25-1.el7.x86_64.rpm
   rpm -ivh mysql-community-client-5.7.25-1.el7.x86_64.rpm
   yum install net-tools
   rpm -ivh mysql-community-server-5.7.25-1.el7.x86_64.rpm
   ```

   需要按照以上安装顺序进行安装，不然会出现依赖问题

   可以使用 `yum update` 升级现有软件及系统内核

4. 启动MySQL

   ```
   systemctl status mysqld (查看MySQL服务状态)
   systemctl start mysqld (启动MySQL服务)
   systemctl enable mysqld (开机启动MySQL服务)
   
   netstat -tunlp (查看已经启动的服务)
   netstat -tunlp | grep mysql 
   
   ps -ef | grep mysql (查看MySQL进程)
   ```

5. 登录MySQL

   MySQL安装会设置临时的密码，需要查看临时密码用来登录MySQL

   通过rpm安装，MySQL是安装到了固定的位置，所以下列命令可以直接使用

   ```
   cat /var/log/mysqld.log (查看此文件的所有内容)
   cat /var/log/mysqld.log | grep password (查看此文件包含password的行信息)
   ```

   ![image-20220616121712150](https://gitee.com/walls1717/images/raw/master/image-20220616121712150.png)

   获取到临时密码后，根据临时密码登录MySQL，并修改密码

   ```
   mysql -uroot -p (使用临时密码登录MySQL)
   
   # 修改密码
   set global validate_password_length=4; (设置密码长度最低4位)
   set global validate_password_policy=LOW; (设置密码安全等级为低，便于设置简单密码)
   set password=password('walls1717'); (设置密码为walls1717)
   
   # 开启访问权限
   grant all on *.* to 'root'@'%' identified by 'walls1717'; ('walls1717'是远程连接所用到的密码)
   flush privileges;
   ```

   如果防火墙没有关闭，而是端口开放，这里就要开放3306端口，MySQL默认为3306端口

   ```
   firewall-cmd --zone=public --add-port=3306/tcp --permanent
   firewall-cmd --reload
   ```



## 3.1.主从配置

详情见目录下 [MySQL主从配置](MySQL主从配置.md) 



# 4.Git

直接通过 `yum install git` 安装即可



# 5.Maven

```
tar -zxvf Maven安装包名 -C /usr/local
vim /etc/profile 修改配置文件，加入如下内容
export MAVEN_HOME=/usr/local/Maven安装路径
export PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$PATH

source /etc/profile
mvn -version
vim /usr/local/Maven安装路径/conf/settings.xml 修改配置文件
<localRepository>/usr/local/repo</localRepository> 设置本地仓库

配置镜像源
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>nexus aliyun</name>
    <url>https://maven.aliyun.com/repository/central</url>
</mirror>
```



# 6.Redis

1. 上传tar.gz到Linux中
2. 解压tar.gz `tar -zxvf redis-4.0.0.tar.gz -C /usr/local` 
3. 安装 gcc `yum install gcc-c++` 
4. 进入 /usr/local/redis-4.0.0 输入命令 `make` 进行编译
5. 进入redis的src目录，输入命令 `make install` 安装

开启redis后台运行

```
/usr/local/redis-4.0.0 进入redis的目录
vim redis.conf 编辑redis配置文件
输入/dae搜索找到 daemonize 将后面改为yes
```

![image-20220616194459628](https://gitee.com/walls1717/images/raw/master/image-20220616194459628.png)

设置redis密码：

```
/usr/local/redis-4.0.0 进入redis的目录
vim redis.conf 编辑redis配置文件
输入/pass搜索找到 requirepass 取消注释，在后面添加密码
```

![image-20220616194302385](https://gitee.com/walls1717/images/raw/master/image-20220616194302385.png)

开启redis远程连接

```
相同的进入redis.conf的编辑页面
输入/bind找到 bind 127.0.0.1 将其注掉即可
```



![image-20220616194549291](https://gitee.com/walls1717/images/raw/master/image-20220616194549291.png)

启动redis

```
进入到src目录下
输入命令./redis-server ../redis.conf (后面的../redis.conf是指定配置文件，因为配置文件在上级目录，所以要有../)
```



# 7.Nginx

1. `yum -y install gcc pcre-devel zlib-devel openssl openssl-devel` (安装环境)
2. 将tar.gz上传到虚拟机
3. `tar -zxvf nginx-1.16.1.tar.gz` (解压)
4. `mkdir /usr/local/nginx` (创建nginx的目录) 
5. 进入到解压之后的文件夹执行 `./configure --prefix=/usr/local/nginx/`
6. 再执行 `make && make install`
