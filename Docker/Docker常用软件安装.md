# Docker常用软件安装

总体步骤：

1. 搜索镜像
2. 拉取镜像
3. 查看镜像
4. 启动镜像 --服务端口映射 `-p`
5. 停止容器
6. 移除容器





# 1. tomcat安装

**不建议使用最新版**

![image-20220703025247404](img.assets\image-20220703025247404.png)

启动tomcat：`docker run -d -p 8080:8080 --name="tomcat" tomcat`

访问若出现404，则进入tomcat容器的操作界面中

![image-20220703030047186](img.assets\image-20220703030047186.png)

**先删除webapps，再将webapps.dist修改为webapps**

再重新启动

![image-20220703030416053](img.assets\image-20220703030416053.png)



# 2. MySQL安装

![image-20220705003642093](img.assets\image-20220705003642093.png)

启动mysql，密码自定义

```shell
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root --name="mysql" mysql:5.7.30
```

![image-20220705004559824](img.assets\image-20220705004559824.png)

进入mysql

```shell
docker exec -it fd0fec968008 mysql -u root -p
```

![image-20220705004856696](img.assets\image-20220705004856696.png)

dokcer容器中mysql默认编码问题:**docker容器中默认使用latin1编码，若在docker容器中直接建表，插入中文会报错**

```mysql
SHOW VARIABLES LIKE 'character%';
```

![image-20220705005617609](img.assets\image-20220705005617609.png)



## 2.1 使用容器数据卷解决字符编码和数据备份

使用容器数据卷同步设置

`docker run -d -p 3306:3306 --privileged=true -v /root/mysql/log:/var/log/mysql -v /root/mysql/data:/var/lib/mysql -v /root/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=root -e TZ=Asia/Shanghai --name mysql mysql:5.7.30`

```shell
docker run -d -p 3306:3306 --privileged=true 
-v /root/mysql/log:/var/log/mysql  
-v /root/mysql/data:/var/lib/mysql  #数据备份
-v /root/mysql/conf:/etc/mysql/conf.d #字符配置
-e MYSQL_ROOT_PASSWORD=root  
-e TZ=Asia/Shanghai
--name mysql mysql:5.7.30  
```

在` /root/mysql/conf`目录下 新建文件`my.cnf`并且配置如下信息:

```
[mysql]
default_character_set=utf8mb4
[mysqld]
collation_server=utf8mb4_general_ci
character_set_server=utf8mb4
```

重新启动mysql容器,查看字符编码

```mysql
SHOW VARIABLES LIKE 'character%';
```

![image-20220705012346680](img.assets\image-20220705012346680.png)



## 2.2 安装mysql主从复制

### 2.1.1 新建主服务3307

`docker run -d -p 3307:3306 --privileged=true -v /root/mysql/mysql-master/log:/var/log/mysql -v /root/mysql/mysql-master/data:/var/lib/mysql -v /root/mysql/mysql-master/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=root --name mysql-master mysql:5.7.30`

```shell
docker run -d -p 3307:3306 --privileged=true 
-v /root/mysql/mysql-master/log:/var/log/mysql 
-v /root/mysql/mysql-master/data:/var/lib/mysql 
-v /root/mysql/mysql-master/conf:/etc/mysql/conf.d 
-e MYSQL_ROOT_PASSWORD=root 
--name mysql-master mysql:5.7.30
```

进入`/root/mysql/mysql-master/conf`创建`my.cnf`文件，并且输入以下内容

```
[mysql]
default_character_set=utf8mb4
[mysqld]

#设置字符编码
collation_server=utf8mb4_general_ci
character_set_server=utf8mb4

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

重启`mysql-master`实例

```shell
docker restart mysql-master
```

进入`mysql-master`实例中创建数据同步用户

```mysql
CREATE USER 'slave'@'%' IDENTIFIED BY '123456'; #创建用户和密码
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%'; #用户授权
```

![image-20220706005529504](img.assets\image-20220706005529504.png)



### 2.1.2 新建从服务3308

`docker run -d -p 3308:3306 --privileged=true -v /root/mysql/mysql-slave/log:/var/log/mysql -v /root/mysql/mysql-slave/data:/var/lib/mysql -v /root/mysql/mysql-slave/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=root --name mysql-slave mysql:5.7.30`

```shell
docker run -d -p 3307:3306 --privileged=true 
-v /root/mysql/mysql-slave/log:/var/log/mysql 
-v /root/mysql/mysql-slave/data:/var/lib/mysql 
-v /root/mysql/mysql-slave/conf:/etc/mysql/conf.d 
-e MYSQL_ROOT_PASSWORD=root 
--name mysql-slave mysql:5.7.30
```

进入`/root/mysql/mysql-slave/conf`创建`my.cnf`文件，并且输入以下内容

```
[mysql]
default_character_set=utf8mb4
[mysqld]

#设置字符编码
collation_server=utf8mb4_general_ci
character_set_server=utf8mb4

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

重启`mysql-slave`实例

```shell
docker restart mysql-slave
```

在**主数据库**中查看主从同步状态

```mysql
show master status;
```

![image-20220706010434597](img.assets\image-20220706010434597.png)

进入`mysql-slave`容器中进行主从配置,**其中的用户名和密码要和主机中创建的同步用户相同**

```mysql
change master to master_host='宿主机ip', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000001', master_log_pos=617, master_connect_retry=30;
```

![image-20220706010847103](img.assets\image-20220706010847103.png)

master_host：主数据库的IP地址；

master_port：主数据库的运行端口；

master_user：在主数据库创建的用于同步数据的用户账号；

master_password：在主数据库创建的用于同步数据的用户密码；

master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；

master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；

master_connect_retry：连接失败重试的时间间隔，单位为秒。



**在数据库中查看主从同步状态**

```mysql
show slave status [\G];
```

![image-20220706011213225](img.assets\image-20220706011213225.png)

在**从数据库**中开启主从同步

```mysql
start slave;
```

![image-20220706011338255](img.assets\image-20220706011338255.png)

查看主从状态

```mysql
show slave status [\G];
```

![image-20220706011530207](img.assets\image-20220706011530207.png)

测试



# 3. Redis安装

对于redis一定要指定配置文件

![image-20220705013203335](img.assets\image-20220705013203335.png)

在宿主机上创建redis.conf原文件并且进行修改，然后使用容器数据卷同步到容器中

![image-20220705013940336](img.assets\image-20220705013940336.png)

![image-20220705014213901](img.assets\image-20220705014213901.png)



**按照指定的redis.conf文件运行redis**

`docker run -p 6379:6379 --name redis --privileged=true -v /root/redis/redis.conf:/etc/redis/redis.conf -v /root/redis/data:/data -d redis:6.2.6 redis-server /etc/redis/redis.conf`

```shell
docker run -p 6379:6379 --name redis --privileged=true 
-v /root/redis/redis.conf:/etc/redis/redis.conf 
-v /root/redis/data:/data 
-d redis:6.2.6 redis-server /etc/redis/redis.conf
```



## 3.1 Redis集群配置

关闭防火墙或者开放端口，新建6个docker容器redis实例

![image-20220706021959728](img.assets\image-20220706021959728.png)

`docker run -d --name redis-node-1 --net host --privileged=true -v /root/redis/share/redis-node-1:/data redis:6.2.6 --cluster-enabled yes --appendonly yes --port 6381`

`docker run -d --name redis-node-2 --net host --privileged=true -v /root/redis/share/redis-node-2:/data redis:6.2.6 --cluster-enabled yes --appendonly yes --port 6382`

`docker run -d --name redis-node-3 --net host --privileged=true -v /root/redis/share/redis-node-3:/data redis:6.2.6 --cluster-enabled yes --appendonly yes --port 6383` 

`docker run -d --name redis-node-4 --net host --privileged=true -v /root/redis/share/redis-node-4:/data redis:6.2.6 --cluster-enabled yes --appendonly yes --port 6384`

`docker run -d --name redis-node-5 --net host --privileged=true -v /root/redis/share/redis-node-5:/data redis:6.2.6 --cluster-enabled yes --appendonly yes --port 6385`

`docker run -d --name redis-node-6 --net host --privileged=true -v /root/redis/share/redis-node-6:/data redis:6.2.6 --cluster-enabled yes --appendonly yes --port 6386`

**命令解释**

· docker run --- 创建并运行docker容器实例

· --name redis-node-6 -- 容器名字

· --net host --- 使用宿主机的IP和端口，默认

· --privileged=true ---- 获取宿主机root用户权限

· -v /root/redis/share/redis-node-6:/data ---容器卷，宿主机地址:docker内部地址

· redis:6.2.6 --- redis镜像和版本号

· --cluster-enabled yes --- 开启redis集群

· --appendonly yes --- 开启持久化

· --port 6386 -- redis端口号

![image-20220706154914811](img.assets\image-20220706154914811.png)



### 3.1.1 构建主从关系

```shell
docker exec -it redis-node-1 /bin/bash
```

进入docker容器后才能执行一下命令，且注意自己的真实IP地址

`redis-cli --cluster create 192.168.153.140:6381 192.168.153.140:6382 192.168.153.140:6383 192.168.153.140:6384 192.168.153.140:6385 192.168.153.140:6386 --cluster-replicas 1`

```shell
redis-cli --cluster create 
192.168.153.140:6381 192.168.153.140:6382 
192.168.153.140:6383 192.168.153.140:6384 
192.168.153.140:6385 192.168.153.140:6386 
--cluster-replicas 1
```

--cluster-replicas 1 表示为每个master创建一个slave节点

![image-20220706164413596](img.assets\image-20220706164413596.png)

![image-20220706164704868](img.assets\image-20220706164704868.png)

哈希槽分配:

![image-20220706170741642](img.assets\image-20220706170741642.png)



### 3.1.2 查看集群状态

链接进入6381作为切入点，查看节点状态

```shell
redis-cli -p 6381
```

`cluster info`:

![image-20220706170032050](img.assets\image-20220706170032050.png)

`cluster nodes`:

![image-20220706170128203](img.assets\image-20220706170128203.png)



### 3.1.3 集群读写error说明

单机环境下`redis-cli -p 6381`进行读写**若值不在对应的哈希槽中将无法进行读写**

![image-20220706171623545](img.assets\image-20220706171623545.png)

加入` -c` 参数采用集群策略连接，设置数据会**自动切换到相应的写主机**

`redis-cli -p 6381 -c`   Redirected --- 重定向 

![image-20220706172135497](img.assets\image-20220706172135497.png)

**添加数据之后，自动切换到对应的主机上**



### 3.1.4 查看集群信息

`redis-cli --cluster check 192.168.153.140:6381`

![image-20220706172802757](img.assets\image-20220706172802757.png)



## 3.2 主从容错切换迁移

先停止6381主机,查看集群状态 

![image-20220706175327493](img.assets\image-20220706175327493.png)

`cluster nodes`:当6381停止后，6385代替6381称为新的主机

![image-20220706175612884](img.assets\image-20220706175612884.png)

再将6381启动，查看集群状态

![image-20220706175842443](img.assets\image-20220706175842443.png)

`cluster nodes`:当6381重启后，**6385仍然是主机，6381成为了从机，主从互换**

![image-20220706175947635](img.assets\image-20220706175947635.png)

数据情况:从主机6385上读取数据

![image-20220706180152035](img.assets\image-20220706180152035.png)



## 3.3  主从扩容

### 3.3.1 新增6387，6388两个节点

将三主三从变为四主四从

`docker run -d --name redis-node-7 --net host --privileged=true -v /root/redis/share/redis-node-7:/data redis:6.2.6 --cluster-enabled yes --appendonly yes --port 6387`

`docker run -d --name redis-node-8 --net host --privileged=true -v /root/redis/share/redis-node-8:/data redis:6.2.6 --cluster-enabled yes --appendonly yes --port 6388`

![image-20220706181510745](img.assets\image-20220706181510745.png)

进入6387容器实例内部`docker exec -it redis-node-7 bash`，将新增的6387作为master节点加入集群

```shell
redis-cli --cluster add-node 192.168.153.140:6387 192.168.153.140:6381
```

6387 就是将要作为master新增节点,6381 就是原来集群节点里面的领路人，相当于6387拜拜6381的码头从而找到组织加入集群

![image-20220706182057216](img.assets\image-20220706182057216.png)

查看集群的情况

```shell
redis-cli --cluster check 192.168.153.140:6381
```

![image-20220706182410795](img.assets\image-20220706182410795.png)

### 3.3.2 重新分配哈希槽

命令:`redis-cli --cluster reshard IP地址:端口号`

```shell
redis-cli --cluster reshard 192.168.153.140:6381
```

![image-20220706182840985](img.assets\image-20220706182840985.png)

**填写要分配哈希槽的节点ID**

![image-20220706183059912](img.assets\image-20220706183059912.png)

**选择全部重新分配**

![image-20220706183325480](img.assets\image-20220706183325480.png)

再次查看集群的情况

```shell
redis-cli --cluster check 192.168.153.140:6381
```

![image-20220706183541161](img.assets\image-20220706183541161.png)

为什么6387是3个新的区间，以前的还是连续？

**重新分配成本太高**，所以前3家各自匀出来一部分，**从6381/6382/6383三个旧节点分别匀出1364个坑位给新节点6387**



### 3.3.3 添加6388为从机

命令：`redis-cli --cluster add-node ip:新slave端口 ip:新master端口 --cluster-slave --cluster-master-id 新主机节点ID`

```shell
redis-cli --cluster add-node 192.168.153.140:6388 192.168.153.140:6387 --cluster-slave --cluster-master-id dbccd3b554a5243a86e025c03c0863dbc24e5353 #新主机节点根据实际来确定
```

![image-20220706184204229](img.assets\image-20220706184204229.png)

再次查看集群情况

```shell
redis-cli --cluster check 192.168.153.140:6381
```

![image-20220706184428318](img.assets\image-20220706184428318.png)



## 3.4 主从缩容

### 3.4.1 删除从机6388

查询从机6388的id

```shell
redis-cli --cluster check 192.168.153.140:6381
```

![image-20220706185618587](img.assets\image-20220706185618587.png)

删除从机6388

命令：`redis-cli --cluster del-node ip:从机端口 从机6388节点ID`

```shell
redis-cli --cluster del-node 192.168.153.140:6388 a92d4fa91e0833b4d95d3cd7378495a7b12d339c
```

![image-20220706185740583](img.assets\image-20220706185740583.png)

查看集群状态:从机6388已经被删除

```shell
redis-cli --cluster check 192.168.153.140:6381
```

![image-20220706185840349](img.assets\image-20220706185840349.png)



### 3.4.2 清空6387哈希槽，进行重新分配

将6387的的哈希槽全部交给6381

```shell
redis-cli --cluster reshard 192.168.153.140:6381
```

![image-20220706190727603](img.assets\image-20220706190727603.png)

再次查看集群状态,已经完成分配

```shell
redis-cli --cluster check 192.168.153.140:6381
```

![image-20220706191152169](img.assets\image-20220706191152169.png)

### 3.4.3 删除主机6387

命令：`redis-cli --cluster del-node ip:端口 6387节点ID`

```shell
redis-cli --cluster del-node 192.168.153.140:6387 dbccd3b554a5243a86e025c03c0863dbc24e5353
```

![image-20220706191432649](img.assets\image-20220706191432649.png)

再次查看集群状态

```shell
redis-cli --cluster check 192.168.153.140:6381
```

![image-20220706191508328](img.assets\image-20220706191508328.png)



# 4.MQ安装

## 4.1.RabbitMQ

### 4.1.1 直接拉取镜像，无managment

```
docker pull rabbitmq
```

> 创建容器

```shell
docker run -d \
--name rabbitmq \
-p 5672:5672 \
-p 15672:15672 \
-v rabbitmq-plugins:/plugins \
-v /root/rabbitmq/data:/var/lib/rabbitmq \
--hostname master \
-e RABBITMQ_DEFAULT_VHOST=master  \
-e RABBITMQ_DEFAULT_USER=root \
-e RABBITMQ_DEFAULT_PASS=root \
rabbitmq:latest
```

说明：
-d：后台运行容器；
–name：指定容器名；
-p：指定服务运行的端口（5672：应用访问端口；15672：控制台Web端口号）；
-v：映射目录或文件；
–hostname：主机名（RabbitMQ的一个重要注意事项是它根据所谓的 “节点名称” 存储数据，默认为主机名）；
-e：指定环境变量；（RABBITMQ_DEFAULT_VHOST：默认虚拟机名；RABBITMQ_DEFAULT_USER：默认的用户名；RABBITMQ_DEFAULT_PASS：默认用户名的密码）
rabbitmq-plugins：表示插件挂载的数据卷，使用命令`docker volume inspect rabbitmq-plugins`可查看其对应的虚拟机目录

> 当然也可以不指定`-e`变量，默认用户名密码都是guest

<hr>

> 现在通过[http://ip:15672](http://ip:15672/)是访问不了的，因为没有web管理插件



### 4.1.2 安装web管理插件`rabbitmq_management`

```shell
docker exec -it rabbitmq rabbitmq-plugins enable rabbitmq_management
```

> `Stats in management UI are disabled on this node`解决

进入容器：

```shell
docker exec -it rabbitmq /bin/bash
```

切到对应目录

```shell
cd /etc/rabbitmq/conf.d/
```

修改` management_agent.disable_metrics_collector = false`

```shell
echo management_agent.disable_metrics_collector = false > management_agent.disable_metrics_collector.conf
```

退出容器

```shell
exit
```

重启容器

```shell
docker restart rabbitmq
```



### 4.1.3 直接拉取镜像，有managment（推荐）

下面命令的冒号后面也可以是对应的版本，比如：`rabbitmq:3.10-management`

```shell
docker pull rabbitmq:management
```

注意：如果`docker pull rabbitmq`后面不带`management`，启动`rabbitmq`后是无法打开管理界面的（跟前面一步一样），所以我们要下载带`management`插件的`rabbitmq`。

```shell
docker run -d \
--name rabbitmq \
-p 5672:5672 \
-p 15672:15672 \
-v rabbitmq-plugins:/plugins \
-v /root/rabbitmq/data:/var/lib/rabbitmq \
-e RABBITMQ_DEFAULT_VHOST=master \
-e RABBITMQ_DEFAULT_USER=root \
-e RABBITMQ_DEFAULT_PASS=13883981813xh \
rabbitmq:management

--hostname master \
```



## 5.Oracle安装

### 5.1获取镜像

```shell
docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```

**或者**

```shell
docker pull registry.aliyuncs.com/helowin/oracle_11g
```



### 5.2 创建并启动容器

**默认启动方式：**

```shell
docker run -itd -p 1521:1521 --name oracle --restart=always registry.aliyuncs.com/helowin/oracle_11g
```

**持久化启动方式**

```shell
docker run  -itd -p 1521:1521 --name oracle --restart=always --mount source=oracle_vol,target=/home/oracle/app/oracle/oradata registry.aliyuncs.com/helowin/oracle_11g
```



### 5.3 进行环境变量配置配置

```shell
docker exec -it oracle bash
```

**切换到root用户**

```shell
su root
```

密码:**helowin**
**配置环境变量, 使用`vi /etc/profile`进行编辑, 末尾加上如下：**

```shell
export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2
export ORACLE_SID=helowin
export PATH=$ORACLEHOME/bin:PATH
```

**保存后执行**

```shell
source /etc/profile 
```



### 5.4 创建软连接

```shell
ln -s $ORACLE_HOME/bin/sqlplus /usr/bin
```

**切换到oracle用户**

```shell
su - oracle
```



### 5.5 sqlplus修改sys、system用户密码

```sql
sqlplus /nolog   --登录
conn /as sysdba
alter user system identified by system ;--修改system用户账号密码；
alter user sys identified by sys ;--修改sys用户账号密码；
```

当执行修改密码的时候出现 ： `database not open`
提示数据库没有打开，不急按如下操作
输入：`alter database open;`
解决办法：
输入：`alter database mount;`
输入 ：`alter database open;`



### 5.6 添加远程登录用户

```sql
create user root identified by root; -- 创建内部管理员账号密码；
grant connect,resource,dba to root; --将dba权限授权给内部管理员账号和密码；
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED; --设置密码永不过期：
alter system set processes=1000 scope=spfile; --修改数据库最大连接数据；
```

>注意:远程登录时
>登录时：SID:helowin
>User: root
>PassWord：root



### 5.7 修改后

```sql
conn /as sysdba;--保存数据库
shutdown immediate; --关闭数据库
startup; --启动数据库
show user;
```

