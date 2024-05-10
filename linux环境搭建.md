# 1.git工具安装

提前安装可能所需要的依赖

```shell
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc-c++ perl-ExtUtils-MakeMaker
```

在 Linux 上安装 Git 向来仅需一行命令即可搞定,当然通过这种方式安装的 Git 可能不是较新版的 Git

```shell
yum install git
```



# 2.JDK安装

1.将JDK8Linux版本的安装包放入/root目录下

2.若系统自带有OpenJDK，先行卸载

```shell
rpm -qa | grep java
```

接下来可以将 java 开头的安装包均卸载即可：

```shell
yum -y remove java-1.7.0-openjdk-1.7.0.141-2.6.10.5.el7.x86_64
yum -y remove java-1.8.0-openjdk-1.8.0.131-11.b12.el7.x86_64
```

3.在 /usr/local/ 下创建 java 文件夹并进入

```shell
cd /usr/local/
mkdir java
cd java
```

4.将上面准备好的 JDK 安装包解压到 /usr/local/java 中即可

```shell
tar -zxvf /root/jdk-8u311-linux-x64.tar.gz -C ./
```

解压完之后， /usr/local/java 目录中会出现⼀个 jdk1.8.0_311 的目录

## 配置JDK环境变量

编辑 **/etc/profile** 文件，在文件尾部加入如下 JDK 环境配置即可

```shell
JAVA_HOME=/usr/local/java/jdk1.8.0_311
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
```

然后执行如下命令让环境变量生效

```shell
source /etc/profile
```

验证java是否安装成功

```shell
java -version
javac
```



# 3.node环境安装

1.将nodeLinux的安装包放入/root目录下 

2.在 /usr/local/ 下创建 node 文件夹并进入

```shell
cd /usr/local/
mkdir node
cd node
```

3.将 Node 的安装包解压到 /usr/local/node 中即可

```shell
 tar -xJvf /root/node-v16.14.0-linux-x64.tar.xz -C ./
```

解压完之后， /usr/local/node 目录中会出现⼀个 node-v16.14.0-linux-x64 的⽬录

## 配置node系统环境变量

1.编辑 **~/.bash_profile** 文件，在文件末尾追加如下信息:

```shell
# Nodejs
export PATH=/usr/local/node/node-v16.14.0-linux-x64/bin:$PATH
```

2.刷新环境变量，使之生效

```shell
source ~/.bash_profile
```

3.检查安装结果

```shell
node -v
npm version
npx -v
```



# 4.python环境安装

CentOS 7版本默认自带了⼀个 Python2.7 环境：

```shell
python
```

然而现在主流都是 Python3 ，所以接下来再装⼀个 Python3 ，打造⼀个共存的环境。



1.准备python3Linux版本安装包，并将其直接放在了 /root 目录下，解压

```shell
tar zxvf Python-3.8.3.tgz
```

2.安装相关预备环境

```shell
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
```

3.指定了安装目录为 /usr/local/python3 ，等安装过程完毕， /usr/local/python3 目录就会生成

```shell
cd Python-3.8.3/
./configure prefix=/usr/local/python3
make && make install
```

4.添加软链接

将刚刚安装生成成的目录 /usr/local/python3 里的 python3 可执行文件做⼀份软链接，链接到 /usr/bin 下，方便后续使用python3

```shell
ln -sf /usr/local/python3/bin/python3 /usr/bin/python3
ln -sf /usr/local/python3/bin/pip3 /usr/bin/pip3
```

5.命令行输入 python3 ，即可查看 Python3 版本的安装结果，而输入 python ，依然还是 python2.7.5 环境

```shell
 python3
```



# 5.maven项目构建和管理工具安装

1.下载 apache-maven-3.8.4-bin.tar.gz 安装包，并将其放置于提前创建好的 /opt/maven 目录

2.执行命令解压，即可在当前目录得到 /opt/maven/apache-maven-3.8.4 目录

```shell
tar zxvf apache-maven-3.8.4-bin.tar.gz
```

3.配置maven加速镜像源

编辑修改 /opt/maven/apache-maven-3.8.4/conf/settings.xml 文件，在 标签对里添加如下内容即可：

```xml
<mirror>
 	<id>alimaven</id>
 	<name>aliyun maven</name>
 	<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
 	<mirrorOf>central</mirrorOf>
</mirror>
```

4.配置环境变量

因为下载的是⼆进制版安装包，所以解压完，配置好环境变量即可使用了。 编辑修改 **/etc/profile** ⽂件，在⽂件尾部添加如下内容，配置 maven 的安装路径

```shell
export MAVEN_HOME=/opt/maven/apache-maven-3.8.4
export PATH=$MAVEN_HOME/bin:$PATH
```

接下来刷新环境变量，让 maven 环境的路径配置生效

```shell
source /etc/profile 
```

5.检验安装结果

执行能打印出 maven 版本信息说明安装、配置成功

```shell
mvn –v 
```



# 6.mysql数据库部署和安装

1.下载mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz 安装包，并将其直接放在了 root 目录

2.卸载系统自带的mariadb（如果有）

首先查询已安装的 Mariadb 安装包：

```shell
rpm -qa|grep mariadb
```

将其均卸载之：

```shell
yum -y remove mariadb-libs-5.5.68-1.el7.x86_64
```

3.将上面准备好的 MySQL 安装包解压到 /usr/local/ 目录，并重命名为 mysql

```shell
tar -zxvf /root/mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz -C /usr/local/
cd /usr/local
mv mysql-5.7.30-linux-glibc2.12-x86_64 mysql
```

4.创建mysql用户和用户组

```shell
groupadd mysql
useradd -g mysql mysql
```

同时新建 /usr/local/mysql/data 目录，后续备用

```shell
cd /usr/local/mysql/
mkdir data
```

5.修改MYSQL目录的归属用户

```shell
chown -R mysql:mysql ./
```

6.在 /etc 目录下新建 my.cnf 文件写入如下简化配置：

```shell
cd /etc
touch my.cnf
```

```shell
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
socket=/var/lib/mysql/mysql.sock
[mysqld]
skip-name-resolve
#设置3306端⼝
port = 3306
socket=/var/lib/mysql/mysql.sock
# 设置mysql的安装⽬录
basedir=/usr/local/mysql
# 设置mysql数据库的数据的存放⽬录
datadir=/usr/local/mysql/data
# 允许最⼤连接数
max_connections=200
# 服务端使⽤的字符集默认为8⽐特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使⽤的默认存储引擎
default-storage-engine=INNODB
lower_case_table_names=1
max_allowed_packet=16M
explicit_defaults_for_timestamp=true
```

7.使用如下命令创建 /var/lib/mysql 目录，并修改权限

```shell
mkdir /var/lib/mysql
chmod 777 /var/lib/mysql
```

8.执行如下命令正式开始安装：**记住随机生成的密码** 

```shell
cd /usr/local/mysql
```

```shell
./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```

若出现以下问题

```shell
./mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
```

**解决方法：**

出现该问题首先检查该 **链接库文件** 有没有安装， 使用命令进行核查。

```shell
rpm -qa | grep libaio   
```

运行该命令后发现系统中无该链接库文件。

使用命令 ：

```shell
yum  install  libaio-devel.x86_64
```

安装成功后，继续运行数据库的初始化命令，提示成功。 <l+.4-PbP2jr

**若依然提示失败，则先删除/var/lib/mysql和 /usr/local/mysql/data两个文件重新生成，在执行初始化**

9.复制启动脚本到资源目录  

```shell
cp ./support-files/mysql.server  /etc/init.d/mysqld
```

并修改 /etc/init.d/mysqld ，修改其 basedir 和 datadir 为实际对应目录：

```shell
vim /etc/init.d/mysqld

basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
```

10.设置mysql系统服务并开启自启

首先增加 mysqld 服务控制脚本执行权限：

```shell
chmod +x /etc/init.d/mysqld
```

同时将 mysqld 服务加入到系统服务：

```shell
chkconfig --add mysqld
```

最后检查 mysqld 服务是否已经生效即可：

```shell
chkconfig --list mysqld
```

在2、3、4、5运行级别随系统启动而自动启动，以后可以直接使用 service 命令控制 mysql 的启停。

11.启动mysql

直接执行

```shell
service mysqld start
```

12.将 mysql的 BIN 目录加入 PATH 环境变量

这样方便便以后在任意目录上都可以使用mysql 提供的命令

编辑 **~/.bash_profile** 文件，在文件末尾处追加如下信息:

```shell
export PATH=$PATH:/usr/local/mysql/bin
```

最后执行如下命令使环境变量生效

```shell
source ~/.bash_profile
```

13.首次登陆mysql

以 root 账户登录 mysql ，使用上文安装完成提示的密码进行登入 

```shell
mysql -u root -p
```

14.修改ROOT账户密码

```mysql
mysql>alter user user() identified by "root";
```

```mysql
mysql>flush privileges;
```

15.设置远程主机登录

```mysql
mysql> use mysql;
mysql> update user set user.Host='%' where user.User='root';
mysql> flush privileges;
```

16.最后利用NAVICAT等工具进行测试

**若云服务器无法连接成功，请在云服务器的安全组中加入3306端口**



# 7.redis缓存安装部署

1.下载 redis-6.2.6.tar.gz 安装包，并将其直接放在了root 目录下

2.在 /usr/local/ 下创建 redis 文件夹并进入

```shell
cd /usr/local/
mkdir redis
cd redis
```

3.、将 Redis 安装包解压到 /usr/local/redis 中

```shell
tar zxvf /root/redis-6.2.6.tar.gz -C ./
```

解压完之后， /usr/local/redis 目录中会出现⼀个 redis-6.2.6 的目录

4.编译并安装

```shell
cd redis-6.2.6/
make && make install
```

5.将 redis安装为系统服务并后台启动

进入utils目录，并执行如下脚本,全部选择默认即可

```shell
[root@localhost redis-6.2.6]# cd utils/
[root@localhost utils]# ./install_server.sh
```

若执行时发生以下错误

```shell
This systems seems to use systemd.
Please take a look at the provided example service unit files in this directory, and adapt and install them. Sorry!
```

**解决方案**：

```shell
vi ./install_server.sh
```

注释下面的代码即可:

```shell
#bail if this system is managed by systemd
#_pid_1_exe="$(readlink -f /proc/1/exe)"
#if [ "${_pid_1_exe##*/}" = systemd ]
#then
#       echo "This systems seems to use systemd."
#       echo "Please take a look at the provided example service unit files in this directory, and adapt and install them. Sorry!"
#       exit 1
#fi
```

6.查看redis服务启动情况

```shell
systemctl status redis_6379.service
```

7.启动redis客户端并测试，启动自带的 redis-cli 客户端

8.设置允许远程连接	

编辑 redis 配置文件

```shell
vim /etc/redis/6379.conf
```

**将 bind 127.0.0.1 修改为 0.0.0.0**

然后重启 Redis 服务

```shell
systemctl restart redis_6379.service
```

**若使用云服务器，在云服务器的安全组中开放6379端口**

9.设置访问密码(如果需要)

编辑 redis 配置文件

```shell
vim /etc/redis/6379.conf
```

找到如下内容：

```shell
#requirepass foobared
```

去掉注释，将 foobared 修改为自己想要的密码，保存即可。

```shell
requirepass root
```

保存，重启 Redis 服务即可

```shell
systemctl restart redis_6379.service
```



#  8.应用服务器tomcat安装部署

1.下载apache-tomcat-9.0.55.tar.gz ，直接将其放在了 /root 目录下

2.在 /usr/local/ 下创建 tomcat 文件夹并进入

```shell
cd /usr/local/
mkdir tomcat
cd tomcat
```

3、将 Tomcat 安装包解压到 /usr/local/tomcat 中

```shell
tar -zxvf /root/apache-tomcat-9.0.55.tar.gz -C./
```

解压完之后， /usr/local/tomcat 目录中会出现⼀个 apache-tomcat-9.0.55 的目录

4.启动Tomcat

直接进 apache-tomcat-9.0.55 目录，执行其中 bin 目录下的启动脚本即可

```shell
[root@localhost apache-tomcat-9.0.55]# cd bin/
[root@localhost bin]# ./startup.sh
```

这时候浏览器访问： **你的主机IP:8080**验证是否启动成功

**若使用云服务器，在云服务器的安全组中开放8080端口**

5.配置快捷操作和开机启动

进入tomcat 的bin目录下的setclasspath.sh文件

```shell
cd /usr/local/tomcat/apache-tomcat-9.0.55/bin
vim setclasspath.sh
```

在开头设置JDK和JRE的位置：

```shell
export JAVA_HOME=/usr/local/java/jdk1.8.0_311
export JRE_HOME=/usr/local/java/jdk1.8.0_311/jre
```

进入 /etc/rc.d/init.d 目录，创建⼀个名为 tomcat 的文件，并赋予执行权限

```shell
[root@localhost ~]# cd /etc/rc.d/init.d/
[root@localhost init.d]# touch tomcat
[root@localhost init.d]# chmod +x tomcat
```

接下来编辑 tomcat 文件，并在其中加入如下内容：

```shell
#!/bin/bash
#chkconfig:- 20 90
#description:tomcat
#processname:tomcat
TOMCAT_HOME=/usr/local/tomcat/apache-tomcat-9.0.55
case $1 in
 		start) su root $TOMCAT_HOME/bin/startup.sh;;
 		stop) su root $TOMCAT_HOME/bin/shutdown.sh;;
 		*) echo "require start|stop" ;;
esac
```

这样后续对于Tomcat的开启和关闭只需要执行如下命令即可：

```shell
service tomcat start
service tomcat stop
```

最后加入开机启动即可：

```shell
chkconfig --add tomcat
chkconfig tomcat on
```



# 9.WEB服务器nginx安装部署

1.下载nginx-1.12.2.tar.gz 安装包，并将其直接放在了 root 目录下

2.在 /usr/local/ 下创建 nginx 文件夹并进入

```shell
cd /usr/local/
mkdir nginx
cd nginx
```

3.将 Nginx 安装包解压到 /usr/local/nginx 中即可

```shell
[root@localhost nginx]# tar zxvf /root/nginx-1.12.2.tar.gz -C ./
```

解压完之后， /usr/local/nginx ⽬录中会出现⼀个 nginx-1.12.2 的目录

4.预先安装额外的依赖

```shell
yum -y install pcre-devel
yum -y install openssl openssl-devel
```

5.编译安装nginx

```shell
cd nginx-1.12.2 
./configure
make && make install
```

安装完成后，Nginx的可执行文件位置位于

```shell
/usr/local/nginx/sbin/nginx
```

6.启动nginx

直接执行如下命令即可：

```shell
[root@localhost sbin]# /usr/local/nginx/sbin/nginx
```

如果想停止Nginx服务，可执行：

```shell
/usr/local/nginx/sbin/nginx -s stop
```

如果修改了配置文件件后想重新加载Nginx，可执行：

```shell
/usr/local/nginx/sbin/nginx -s reload
```

注意其配置文件位于

```shell
/usr/local/nginx/conf/nginx.conf
```

7.浏览器验证启动情况:http://IP地址



# 10.zookeeper安装部署

1.下载 apache-zookeeper-3.5.7-bin.tar.gz 压缩包，并将其放在 /root 目录下

2.在 /usr/local/ 下创建 zookeeper文件夹并进入

```shell
cd /usr/local/
mkdir zookeeper
cd zookeeper
```

3.将 ZooKeeper 安装包解压到 /usr/local/zookeeper 中即可

```shell
[root@localhost zookeeper]# tar -zxvf /root/apache-zookeeper-3.5.7-bin.tar.gz -C ./
```

解压完之后， /usr/local/zookeerper 目录中会出现⼀个 apache-zookeeper-3.5.7-bin 的目录

4.创建data目录

直接在 /usr/local/zookeeper/apache-zookeeper-3.5.7-bin 目录中创建**⼀个 data 目录**,等下该 data 目录地址要配到 ZooKeeper 的配置文件中

5.创建配置文件并修改

进入到 zookeeper 的 conf 目录，复制 zoo_sample.cfg 得到 zoo.cfg

```shell
[root@localhost apache-zookeeper-3.5.7-bin]# cd conf/
[root@localhost conf]# cp zoo_sample.cfg zoo.cfg
```

修改配置文件 zoo.cfg ，将其中的 dataDir 修改为上面刚创建的 data 目录，其他选项可以按需配置

```shell
vim zoo.cfg
```

```shell
dataDir=/usr/local/zookeeper/apache-zookeeper-3.5.7-bin/data
```

6.启动zookeeper

```shell
[root@localhost apache-zookeeper-3.5.7-bin]# ./bin/zkServer.sh start
```

启动后可以通过如下命令来检查启动后的状态：**zookeeper默认会绑定端口2181**

```shell
[root@localhost apache-zookeeper-3.5.7-bin]# ./bin/zkServer.sh status
```

若出现以下错误：

```shell
Error contacting service. It is probably not running.
```

是因为**zookeeper3.5.X默认占用8080端口**，**需要在 zoo.cfg文件中添加**：

```shell
admin.serverPort=99
```

7.配置环境变量

编辑配置文件：

```shell
vim /etc/profile
```

尾部加入ZooKeeper 的 bin 路径配置即可:

```shell
export ZOOKEEPER_HOME=/usr/local/zookeeper/apache-zookeeper-3.5.7-bin
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

最后执行使环境变量生效即可

```shell
source /etc/profile 
```

8.设置开机自启

首先进入 /etc/rc.d/init.d 目录，创建⼀个名为 zookeeper 的文件，并赋予执行权限

```shell
[root@localhost ~]# cd /etc/rc.d/init.d/
[root@localhost init.d]# touch zookeeper
[root@localhost init.d]# chmod +x zookeeper
```

编辑 zookeeper 文件，并在其中加入如下内容：

```shell
#!/bin/bash
#chkconfig:- 20 90
#description:zookeeper
#processname:zookeeper
ZOOKEEPER_HOME=/usr/local/zookeeper/apache-zookeeper-3.5.7-bin
export JAVA_HOME=/usr/local/java/jdk1.8.0_311 # 此处根据你的实际情况更换对应
case $1 in
 start) su root $ZOOKEEPER_HOME/bin/zkServer.sh start;;
 stop) su root $ZOOKEEPER_HOME/bin/zkServer.sh stop;;
 status) su root $ZOOKEEPER_HOME/bin/zkServer.sh status;;
 restart) su root $ZOOKEEPER_HOME/bin/zkServer.sh restart;;
 *) echo "require start|stop|status|restart" ;;
esac
```

加入开机启动

```shell
chkconfig --add zookeeper
chkconfig zookeeper on
```





# 11.RabbitMQ安装部署

因为 RabbitMQ 需要 erlang 环境的支持，所以必须先安装 erlang 。

我们这里要安装的是 erlang-22.3.3-1.el7.x86_64.rpm ，所以⾸先执⾏如下命令来安装其对应的yum repo ：

```shell
curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash
```

接下来执行如下命令正式安装 erlang 环境：

```shell
yum install erlang-22.3.3-1.el7.x86_64
```

出现提示则执行下面命令：

![image-20220317195158549](img.assets\image-20220317195158549.png)

```shell
yum load-transaction /tmp/yum_save_tx.2020-05-14.22-21.n0cwzm.yumtx
```

接下来可以直接执行如下命令，测试 erlang 是否安装成功：

```shell
erl
```

接下来正式开始安装 rabbitmq ，首先先依然是安装其对应的 yum repo ：

```shell
curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
```

然后执行如下命令正式安装 rabbitmq ：

```shell
yum install rabbitmq-server-3.8.3-1.el7.noarch
```

启动RabbitMQ服务

```shell
systemctl start rabbitmq-server.service
```

启动服务

```shell
 /sbin/service rabbitmq-server start 
```

查看服务状态

```shell
 /sbin/service rabbitmq-server status
```

停止服务(选择执行)

```shell
/sbin/service rabbitmq-server stop
```

设置RabbitMQ开机启动

```shell
systemctl enable rabbitmq-server.service
```

开启web可视化管理插件：

```shell
rabbitmq-plugins enable rabbitmq_management
```

若出现

```
 Plugin configuration unchanged.
```

可以重新启用

```
rabbitmq-plugins disable rabbitmq_management
rabbitmq-plugins enable rabbitmq_management
```

访问可视化管理界面 ：若使用云服务器，则在安全组中开发15672端口

```
浏览器输⼊： 你的服务器IP:15672
```

![image-20220317201035558](img.assets\image-20220317201035558.png)

我们可以在后台先添加⼀个用户/密码对：

```shell
rabbitmqctl add_user root root
# 将账号设置为超级管理员
rabbitmqctl set_user_tags root administrator
#设置用户权限
rabbitmqctl set_permissions -p "/" root ".*" ".*" ".*"
```

若出现错误

```
Error: unable to perform an operation on node 'rabbit@xh'. Please see diagnostics information and suggestions below.
```

解决方案：

1.查找自己的主机名

```
hostname
```

2.vim /etc/hosts

3.加入自己的ip和主机名 



![image-20220317201308046](img.assets\image-20220317201308046.png)

关闭应用的命令为

```
rabbitmqctl stop_app
```

清除的命令为

```
rabbitmqctl reset
```

重新启动命令为

```
rabbitmqctl start_app
```



# 12.consul安装

```shell
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install consul
```

启动：

```shell
consul agent -dev -client=0.0.0.0 #后台启动
```

若要使用可视化页面，开放8500端口,**页面地址:ip:8500**

相关命令:

```shell
consul leave #关闭节点
consul reload #重新加载配置文件
consul catalog services #查询所有注册的服务
```



# 13.zipkin

安装并且启动

```shell
curl -sSL https://zipkin.io/quickstart.sh | bash -s
java -jar zipkin.jar &
```

**开发端口9411**

可视化页面:ip:9411



# 14.nacos

这里使用：`nacos-server-2.0.3.tar.gz` 放置在在root目录下

1.解压安装包

```shell
tar -zxvf nacos-server-2.0.3.tar.gz
```

2.将解压后的文件夹放到服务器/usr/local文件夹下

```shell
mv nacos /usr/local/
```

3.输入指令，启动nacos

在bin目录下：

```shell
sh startup.sh -m standalone #启动单机版
sh startup.sh #启动集群版
# 关闭nacos
sh shutdown.sh
```

4.启动客户端 **ip:8848/nacos**

默认账号和密码为nacos



5.修改数据库

Nacos默认**自带的是嵌入式数据库derby**

derby到mysql切换配置步骤：

1. usr\local\nacos\conf录下找到nacos-mysql.sql文件，执行脚本。---将sql语句粘贴到本地的mysql中
2. usr\local\nacos\conf目录下找到application.properties，添加以下配置（按需修改对应值）。

```properties
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=root
db.password.0=root
```

​	3.重启



# 15. Sentinel

Sentinel下载后为一个jar包，直接运行即可

[下载地址](https://github.com/alibaba/Sentinel/releases)

- 运行命令
  - 前提
    - Java 8 环境
    - 8080端口不能被占用
  - 命令
    - `java -jar sentinel-dashboard-1.8.4.jar`
- 访问Sentinel管理界面
  - localhost:8080
  - 登录账号密码均为sentinel



指定端口启动

```shell
java -Dserver.port=8080 -jar  #用于指定 Sentinel 控制台端口
```



# 16. seata

下载([seata](https://github.com/seata/seata/releases))`seata-server-1.4.2.tar.gz`

安装到/usr/local/下:

```
cd /usr/local/
```

安装:

```
 tar -zxvf /root/seata-server-1.4.2.tar.gz -C ./
```

安装完毕在seata目录下有一个seata-server-1.4.2



# 7.Hadoop

```shell
tar -zxvf hadoop-3.1.3.tar.gz -C /opt/module/
```

修改配置文件

```shell
vim /etc/profile
```

```sh
#HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

刷新当前的shell环境

```shell
source /etc/profile
```

最后查看是否成功安装

```
hadoop
```
