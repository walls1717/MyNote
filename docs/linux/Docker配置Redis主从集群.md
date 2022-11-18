# 1.主从集群配置

首先启动6台Redis容器，搭建3主3从的主从集群

```dockerfile
# 修改不同的端口与名称，执行6条命令即启动6台Redis容器
docker run -d --net host --privileged=true \
-v /data/redis/share/redis-node-1:/data \
--name redis-node-1 redis:[tag] \
--cluster-enabled yes --appendonly yes \
--port 6381

#部分参数说明
--net host：使用宿主机的IP和端口，默认
--privileged=true：获取宿主机root用户权限
--cluster-enabled yes：开启redis集群
--appendonly yes：开启持久化
--port 6386：redis端口号
```

进入一台Redis容器(无所谓哪一台) `docker exec -it redis-node-1 /bin/bash` 

构建主从关系，出现下图信息则构建成功

```
# 根据自己的配置修改ip与端口，并删掉多余换行执行
redis-cli --cluster create 
192.168.182.110:6381 
192.168.182.110:6382 
192.168.182.110:6383 
192.168.182.110:6384 
192.168.182.110:6385 
192.168.182.110:6386 
--cluster-replicas 1
```

![image-20220918110714973](https://raw.githubusercontent.com/walls1717/image/master/202209181107399.png)

进入Redis客户端查看集群状态 `redis-cli -p 6381` 

```
cluster info
cluster notes
```

集群操作常用命令

```
# 以集群方式进入Redis客户(容器输入)
redis-cli -p 6382 -c
# 查看集群状态(主机输入)
redis-cli --cluster check 自己ip:6381
```



# 2.主从集群扩容

再添加两台Redis容器为扩容做准备(复用之前的命令即可)，我这里新增两台6387和6388

进入6387容器内部 `docker exec -it redis-node-7 /bin/bash` 

将新增的6387节点作为master节点加入原集群 `redis-cli --cluster add-node 自己实际IP地址:6387 自己实际IP地址:6381` 

![image-20220918165611520](https://raw.githubusercontent.com/walls1717/image/master/202209181656929.png)

重新分配槽号 `redis-cli --cluster reshard 192.168.111.147:6381` 这里会输入几条参数，如图

```
# 部分参数说明
4096：16384/master台数
id：6387的节点id(通过check可查询到)
all：使用所有节点作为哈希槽的源节点
```

![image-20220918165808354](https://raw.githubusercontent.com/walls1717/image/master/202209181658823.png)

为主节点分配从节点 `redis-cli --cluster add-node ip:新slave端口 ip:新master端口 --cluster-slave --cluster-master-id 新主机节点ID` 

![image-20220918170316815](https://raw.githubusercontent.com/walls1717/image/master/202209181703685.png)



# 3.主从集群缩容

这里以6387与6388下线作为案例演示

6388作为6387的从节点，首先就要干掉他 `redis-cli --cluster del-node 192.168.182.110:6388 6388的节点id` 

清空6387的槽位号，将槽位号重新分配 `redis-cli --cluster reshard 192.168.111.147:6381` 

```
# 参数说明
4096：需要分配的槽位
id：接手槽位的节点id
node：分配槽位的节点id
```

![image-20220918172137600](https://raw.githubusercontent.com/walls1717/image/master/202209181721200.png)

干掉6387节点 `redis-cli --cluster del-node 192.168.111.147:6387 6387节点id` 