# 1. 准备两台虚拟机

在Vmware中创建两台Linux虚拟机，如果已经配置过一个Linux虚拟机，则可以直接克隆这个虚拟机，克隆完之后修改虚拟机的IP地址，并重启网络服务。

<img src="https://gitee.com/walls1717/images/raw/master/202208311313527.png" alt="image-20220619110259933" style="zoom:50%;" />

# 2. 修改MySQL的UUID

因为是克隆的虚拟机，所以MySQL的UUID也是一致的，这会导致后面启动slave线程失败。

1. 在MySQL中生成UUID

   `select uuid();`

   <img src="https://gitee.com/walls1717/images/raw/master/202208311313969.png" alt="image-20220619110744494" style="zoom: 67%;" />

2. 查看配置文件目录

   `show variables like 'datadir';`

   <img src="https://gitee.com/walls1717/images/raw/master/202208311314024.png" alt="image-20220619110850670" style="zoom:67%;" />

3. 编辑配置文件

   `vim /var/lib/mysql/auto.cnf`

   将UUID替换为之前新生成的UUID

   <img src="https://gitee.com/walls1717/images/raw/master/202208311314304.png" alt="image-20220619110956228" style="zoom:67%;" />

4. 重启服务

   `systemctl restart mysqld`

# 3. 设置主库MySQL

1. 编辑配置文件

   `vim /etc/my.cnf`

   ```
   [mysqld]
   log-bin=mysql-bin # 启用二进制文件
   server-id=100 # 服务器唯一id
   ```

2. 添加新用户用来访问从库MySQL

   `grant replication slave on *.* to 'zhusql'@'%' identified by 'walls1717';`

   这里的用户名和密码都是可以随意更改的。

   如果在执行此sql出现如下报错，就需要修改密码的检查等级。

   ![image-20220619111402342](https://gitee.com/walls1717/images/raw/master/202208311314104.png)

   ```
   set global validate_password_policy=0; (设置密码检查等级为LOW)
   set global validate_password_length=4; (设置密码长度最低为4位)
   show variables like 'validate_password%'; (查看密码检查表，是否设置成功)
   ```

3. 查看二进制文件信息

   `show master status;`

   ![image-20220619111554653](https://gitee.com/walls1717/images/raw/master/202208311314768.png)

   这里要记录下来File和Position的信息，并且不能再对这个MySQL有任何操作，一旦有操作，此信息就会发生变化。

# 4. 设置从库MySQL

1. 编辑配置文件

   `vim /etc/my.cnf`

   ```
   [mysqld]
   server-id=101 #服务器唯一id
   ```



`change master to master_host='192.168.66.11',master_user='zhusql',master_password='walls1717',master_log_file='mysql-bin.000001',master_log_pos=439;`

在从MySQL中执行这条命令，注意其中host，user，password，file，pos都要根据自己的实际情况设置。

接下来执行 `start slave;` 运行slave的IO线程。

执行 `show slave status;` 查看线程是否成功运行，这里如果是YES则运行成功。

![image-20220619111953554](https://gitee.com/walls1717/images/raw/master/202208311314901.png)