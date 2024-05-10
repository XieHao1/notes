# Docker进阶



[TOC]

# 一.Docker file

Dockerfile是用来构建Docker镜像的文本文件，是由一条条构建镜像的指令和参数构成的脚本

[官网](https://docs.docker.com/engine/reference/builder/)

![image-20220706231959473](img.assets\image-20220706231959473.png)



**构建三步骤：**

​	1.编写Dockerfile文件

​	2.`docker bulid`命令构建镜像

​	3.`docker run`依镜像运行容器实例

 

## 1.1 Dockerfile构建解析过程

### 1.1.1 dokcerfile内容基础知识

 1：每条保留字指令都**必须为大写字母**且后面要跟随至少一个参数

 2：指令按照从上到下，顺序执行

 3：#表示注释

 4：每条指令都会创建一个新的镜像层并对镜像进行提交



### 1.1.2 Docker执行Dokcerfile的大致流程

（1）docker从基础镜像运行一个容器

（2）执行一条指令并对容器作出修改

（3）执行类似docker commit的操作提交一个新的镜像层

（4）docker再基于刚提交的镜像运行一个新容器

（5）执行dockerfile中的下一条指令直到所有指令都执行完成



从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段，

\*  Dockerfile是软件的原材料

\*  Docker镜像是软件的交付品

\*  Docker容器则可以认为是软件镜像的运行态，也即依照镜像运行的容器实例

Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

![image-20220706232946987](img.assets\image-20220706232946987.png)

1 Dockerfile，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等;

 

2 Docker镜像，在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时会真正开始提供服务;

 

3 Docker容器，容器是直接提供服务的。



## 1.2 Dokcerfile常用保留字指令

![image-20220707002259785](img.assets\image-20220707002259785.png)

### 1.2.1  FROM

基础镜像，当前新镜像是基于哪个镜像的，指定一个已经存在的镜像作为模板，第一条必须是from

```dockerfile
FROM [--platform=<platform>] <image> [AS <name>]
```

Or

```dockerfile
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
```

Or

```dockerfile
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```



### 1.2.2 MAINTAINER

镜像维护者的姓名和邮箱地址

```dockerfile
MAINTAINER <name>
```



### 1.2.3 RUN

容器构建时需要运行的命令,**RUN是在 docker build时运行**

**RUN的两种格式:**

1.shell格式

```dockerfile
RUN <命令行命令> #<命令行命令>等同于在终端操作的shell命令
```

```dockerfile
RUN yum -y install vim
```

2.exec格式

```dockerfile
RUN ["可执行文件","参数1",“参数2”]
```

```dockerfile
RUN ["/bin/bash", "-c", "echo hello"]
```



### 1.2.4 EXPOSE

当前容器对外暴露出的端口

```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

```dockerfile
EXPOSE 80/udp
EXPOSE 80/tcp
```



### 1.2.5 WORKDIR

指定在创造容器后，终端默认登录的进来的工作目录

```dockerfile
WORKDIR /path/to/workdir
```

```dockerfile
ENV DIRPATH=/path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd
```



### 1.2.6 USER

指定该镜像以什么样的用户去执行，如果不指定，默认是root

```dockerfile
USER <user>[:<group>]
```

or

```dockerfile
USER <UID>[:<GID>]
```

```dockerfile
FROM microsoft/windowsservercore
# Create Windows user in the container
RUN net user /add patrick
# Set it for subsequent commands
USER patrick
```



### 1.2.7 ENV

用来在构建镜像的过程中设置环境变量

```dockerfile
ENV <key>=<value> ...
```

```dockerfile
ENV MY_PATH /usr/mytest
```

这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样；也可以在其它指令中直接使用这些环境变量，比如

**先定义，再引用**

```dockerfile
WORKDIR $MY_PATH
```



### 1.2.8 ADD

将宿主机目录下的文件拷贝进镜像且会自动处理URL和解压tar压缩包

```dockerfile
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

```dockerfile
ADD hom* /mydir/ #添加所有以“hom”开头的文件
ADD hom?.txt /mydir/ #？替换为任何单个字符，例如“home.txt”
ADD test.txt relativeDir/ #<dest> 是绝对路径，或相对于 WORKDIR 的路径，源将在目标容器内复制到其中。使用相对路径，并将“test.txt”添加到 								  #<WORKDIR>/relativeDir/
ADD test.txt /absoluteDir/ #使用绝对路径，并将“test.txt”添加到 /absoluteDir/
ADD arr[[]0].txt /mydir/  #添加包含特殊字符（如 [ 和 ] ）的文件或目录时，您需要按照 Golang 规则对这些路径进行转义，以防止它们被视为匹配模式。
```

```dockerfile
ADD --chown=55:mygroup files* /somedir/
ADD --chown=bin files* /somedir/
ADD --chown=1 files* /somedir/
ADD --chown=10:11 files* /somedir/
```



### 1.2.9 COPY

类似ADD，拷贝文件和目录到镜像中。 将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置

```dockerfile
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

```dockerfile
COPY src dest # <src源路径>：源文件或者源目录  <dest目标路径>：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。
COPY ["src", "dest"]
```

```dockerfile
COPY hom* /mydir/
COPY hom?.txt /mydir/
COPY test.txt relativeDir/
COPY test.txt /absoluteDir/
COPY arr[[]0].txt /mydir/
```

```dockerfile
COPY --chown=55:mygroup files* /somedir/
COPY --chown=bin files* /somedir/
COPY --chown=1 files* /somedir/
COPY --chown=10:11 files* /somedir/
```



### 1.2.10 VOLUME

容器数据卷，用于数据保存和持久化工作

```dockerfile
VOLUME ["/data"]
```

```dockerfile
FROM ubuntu
RUN mkdir /myvol
RUN echo "hello world" > /myvol/greeting
VOLUME /myvol
```



### 1.2.11 CMD

指定容器启动后要干的事情

Dockerfile 中可以有多个 CMD 指令，但**只有最后一个生效，CMD 会被 docker run 之后的参数替换**

1.shell格式

```dockerfile
CMD <命令行命令>
```

2.**exec格式--首选形式**

```dockerfile
CMD ["可执行文件","参数1","参数2"]
```

3.参数列表格式

```dockerfile
CMD ["参数1","参数2"] #再指定的ENTRYPOINT指令后，用CMD指定具体的参数
```

```dockerfile
FROM ubuntu
CMD echo "This is a test." | wc -
```

```dockerfile
FROM ubuntu
CMD ["/usr/bin/wc","--help"]
```



**RUN和CMD的区别:**

CMD是在docker run 时运行。

RUN是在 docker build时运行。



### 1.2.12 ENTRYPOINT

指定一个容器启动时要运行的命令

类似于 CMD 指令，但是ENTRYPOINT**不会被docker run后面的命令覆盖**， 而且这些命令行参数**会被当作参数送给 ENTRYPOINT 指令指定的程序**

1.**exec 形式--首选形式**

```dockerfile
ENTRYPOINT ["executable", "param1", "param2"]
```

2.shell 格式

```dockerfile
ENTRYPOINT command param1 param2
```

ENTRYPOINT 允许配置将作为可执行文件运行的容器。

```shell
$ docker run -i -t --rm -p 80:80 nginx
```



ENTRYPOINT可以和CMD一起用，一般是**变参才会使用 CMD** ，这里的 CMD 等于是在给 ENTRYPOINT 传参。

当指定了ENTRYPOINT后，CMD的含义就发生了变化，不再是直接运行其命令而是**将CMD的内容作为参数传递给ENTRYPOINT指令**

```dockerfile
<ENTRYPOINT><CMD>
```

假设已通过 Dockerfile 构建了 nginx:test 镜像

![image-20220707001955420](img.assets\image-20220707001955420.png)

| 是否传参         | 按照dockerfile编写执行         | 传参运行                                     |
| ---------------- | ------------------------------ | -------------------------------------------- |
| Docker命令       | docker run nginx:test          | docker run nginx:test -c /etc/nginx/new.conf |
| 衍生出的实际命令 | nginx -c /etc/nginx/nginx.conf | nginx -c /etc/nginx/new.conf                 |



## 1.3 编写dockerfile实现自定义镜像

### 1.3.1 自定义镜像centos-jdk8

centos7镜像具备vim+ifconfig+jdk8

**编写Dockerfile文件**

  1.下载jdk8安装包并且放入root中

![image-20220707004116898](img.assets\image-20220707004116898.png)

2.编写**D**ockerfile文件---**大写字母D**

```
vim DockerFile
```

```dockerfile
FROM centos:7
MAINTAINER xh<1693062665@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

#安装vim编辑器
RUN yum -y install vim
#安装ifconfig命令查看网络IP
RUN yum -y install net-tools
#安装java8及lib库
RUN yum -y install glibc.i686
RUN mkdir /usr/local/java
#ADD 是相对路径jar,把jdk-8u311-linux-x64.tar.gz添加到容器中,安装包必须要和Dockerfile文件在同一位置
ADD jdk-8u311-linux-x64.tar.gz /usr/local/java/
#配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_311
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH

EXPOSE 80

CMD echo $MYPATH
CMD echo "success--------------ok"
cMD /bin/bash
```

3.构建

```shell
docker bulid -t 新镜像的名字:TAG . #TAG后面有一个空格和.
```

```shell
docker build -t centos7jdk8:1.0.0 .
```

![image-20220707005845812](img.assets\image-20220707005845812.png)

![image-20220707005904781](img.assets\image-20220707005904781.png)

4.运行

```shell
docker run -it centos7jdk1.8:1.0.0
```

![image-20220707010137307](img.assets\image-20220707010137307.png)

![image-20220707010218581](img.assets\image-20220707010218581.png)



### 1.3.2 自定义Ubuntu镜像

```dockerfile
FROM ubuntu
MAINTAINER xh<1693062665@qq.com>
 
ENV MYPATH /usr/local
WORKDIR $MYPATH
 
RUN apt-get update
RUN apt-get install net-tools
#RUN apt-get install -y iproute2
#RUN apt-get install -y inetutils-ping
 
EXPOSE 80
 
CMD echo $MYPATH
CMD echo "install inconfig cmd into ubuntu success--------------ok"
CMD /bin/bash
```

构建：

```shell
docker build -t my-ubuntu:1.0.0 .
```

运行

```shell
docker run -it  my-ubuntu:1.0.0
```



## 1.4 虚悬镜像

仓库名、标签都是<none>的镜像，俗称dangling image

使用Dockerfile写一个虚悬镜像:

```dockerfile
from ubuntu
CMD echo 'action is success'
```

构建

```shell
docker build .
```

![image-20220707010910056](img.assets\image-20220707010910056.png)

查看:仓库名、标签都是<none>的镜像

```shell
docker image ls -f dangling=true
```

![image-20220707010948946](img.assets\image-20220707010948946.png)

删除:虚悬镜像已经失去存在价值，可以删除

```shell
docker image prune
```

![image-20220707011157643](img.assets\image-20220707011157643.png)



# 二.Dokcer运行微服务

## 2.1 使用maven打一个jar包并且上传

![image-20220707012311046](img.assets\image-20220707012311046.png)

## 2.2 编写Dockerfile

```dockerfile
# 基础镜像使用java
FROM java:8
# 作者
MAINTAINER xh
# VOLUME 指定临时文件目录为/tmp，在主机/var/lib/docker目录下创建了一个临时文件并链接到容器的/tmp
VOLUME /tmp
# 将jar包添加到容器中并更名为readapp.jar
ADD readapp-1.0.0.jar readapp.jar
# 运行jar包
RUN bash -c 'touch /readapp.jar'
ENTRYPOINT ["java","-jar","/readapp.jar"]
#暴露8888端口作为微服务
EXPOSE 8888
```



## 2.3 构建

```shell
docker build -t readapp:1.0.0 .
```

![image-20220707014100527](img.assets\image-20220707014100527.png)

## 2.4 运行容器

```shell
docker run -d -p 8888:8888 readapp:1.0.0 #不需要加java -jar 在Dockerfile文件中已经指定
```

![image-20220707014300456](img.assets\image-20220707014300456.png)



# 三.Docker网络

在不启动docker容器时，默认的网络状况:`ens33` ,`lo`,`virbr0`	

![image-20220721012948116](img.assets\image-20220721012948116.png)

在CentOS7的安装过程中如果有选择相关虚拟化的的服务安装系统后，启动网卡时会发现有一个**以网桥连接的私网地址的virbr0网卡**(virbr0网卡：它还有一个固定的默认IP地址192.168.122.1)，是做虚拟机网桥的使用的，其作用是为连接其上的虚机网卡提供 NAT访问外网的功能。



删除virbr0网卡:

```
yum remove libvirt-libs.x86_64
```



**启动docker时，默认的网络状态:会产生一个名为docker0的虚拟网桥**

![image-20220721013354343](img.assets\image-20220721013354343.png)





## 3.1 docker网络常用命令

所有命令

```shell
docker network COMMAND
```

![image-20220721013838873](img.assets\image-20220721013838873.png)



### 3.1.1 查看docker网络命令

默认创建3大网络模式

```shell
docker network ls
```

![image-20220721013558244](img.assets\image-20220721013558244.png)

### 3.1.2 创建docker网络

```shell
docker network create 网络名
```

![image-20220721014221673](img.assets\image-20220721014221673.png)

### 3.1.3 删除docker网络

```shell
docker network rm 网络名
```

![image-20220721014411652](img.assets\image-20220721014411652.png)



### 3.1.4 查看网络源数据

```shell
docker network inspect XXX网络的名字
```

![image-20220721014526986](img.assets\image-20220721014526986.png)



## 3.2 docker网络的作用

1.**容器间**的互联和通信以及端口映射

2.容器IP变动时候可以通过服务名直接网络通信而不受到影响



## 3.3 docker网络模式

| 网络模式   | 简介                                                         | 指定方式                                 |
| :--------- | :----------------------------------------------------------- | :--------------------------------------- |
| bridge模式 | 为每一个容器分配，设置IP等，并将容器连接到一个`docker0`虚拟网桥，**默认为该模式** | `--network bridge`，默认使用docker0      |
| host模式   | 容器将不会虚拟出自己的网卡，配置自己的ipdeng，而是**使用宿主机的ip和端口** | `--network host`                         |
| none模式   | 容器有独立的network namespace,但没有对其进行任何网络设置，如分配veth pair和网桥连接，ip等。 | `--network none`                         |
| container  | 新创建的容器不会创建自己的网卡和配置自己的IP，而是和一个指定的容器共享IP、端口范围等 | `--network container:NAME`或者容器ID指定 |



### 3.3.1 docker容器实例内默认网络ip产生的规则



**docker容器默认网络ip是有可能发生改变的**



1.先启动两个ubuntu容器实例

```shell
docker run -it --name u1 ubuntu /bin/bash
docker run -it --name u2 ubuntu /bin/bash
```

![image-20220721020504442](img.assets\image-20220721020504442.png)



2.分别查看容器实例内部ip

`docker inspect 容器ID or 容器名字`

```shell
docker inspect u1
docker inspect u2
```

u1内部ip：172.17.0.2

![image-20220721020734331](img.assets\image-20220721020734331.png)

u2内部ip：172.17.0.3

![image-20220721020835704](img.assets\image-20220721020835704.png)



3.关闭容器u2，新建容器u3，再次查看容器ip

```shell
docker stop u2
docker run -it --name u3 ubuntu /bin/bash
docker inspect u3
```

u3内部ip：172.17.0.3,容器ip发生改变

![image-20220721021050520](img.assets\image-20220721021050520.png)



### 3.3.2 bridge模式

Docker 服务默认会创建一个 docker0 网桥（其上有一个 docker0 内部接口），该桥接网络的名称为docker0，它在**内核层**连通了其他的物理或虚拟网卡，这就将所有容器和本地主机都**放到同一个物理网络**。Docker 默认指定了 docker0 接口 的 IP 地址和子网掩码，**让主机和容器之间可以通过网桥相互通信**。



**说明 ：把docker0当作交换机，每个容器占用一个交换机的网口，所有容器进行数据通信交换必须经过这个docker0交换机才可以。**

1. Docker使用Linux桥接，在宿主机虚拟一个Docker容器网桥(docker0)，Docker启动一个容器时会根据Docker网桥的网段分配给容器一个IP地址，称为Container-IP，同时Docker网桥是每个容器的默认网关。因为在同一宿主机内的容器都接入同一个网桥，这样容器之间就能够通过容器的Container-IP直接通信。

2. docker run 的时候，没有指定network的话默认使用的网桥模式就是bridge，使用的就是docker0。在宿主机ifconfig,就可以看到docker0和自己create的network eth0，eth1，eth2……代表网卡一，网卡二，网卡三……，lo代表127.0.0.1，即localhost，inet addr用来表示网卡的IP地址

3. 网桥docker0**创建一对对等虚拟设备接口**一个叫veth，另一个叫eth0，成对匹配。

   3.1 整个宿主机的网桥模式都是docker0，类似一个交换机有一堆接口，每个接口叫veth，在本地主机和容器内分别创建一个虚拟接口，并让他们彼此联通（这样一对接口叫veth pair；

  	  3.2 每个容器实例内部也有一块网卡，每个接口叫eth0；
  	
  	  3.3 docker0上面的每个veth匹配某个容器实例内部的eth0，两两配对，一一匹配。

 通过上述，将宿主机上的所有容器都连接到这个内部网络上，两个容器在同一个网络下,会从这个网关下各自拿到分配的ip，此时两个容器的网络是互通的。

![image-20220721022227816](img.assets\image-20220721022227816.png)



**实例：**

1.启动tomcat容器

```
docker start tomcat9.0.55
```

在宿主机上查看网络关系`ip addr`

![image-20220721023609323](img.assets\image-20220721023609323.png)

2.进入tomcat容器，查看网络关系

```
docker exec -it tomcat9.0.55 bash
```

![image-20220721023954427](img.assets\image-20220721023954427.png)



### 3.3.3 host模式

直接使用宿主机的 IP 地址与外界进行通信，不再需要额外进行NAT 转换



**说明:**

容器将不会获得一个独立的Network Namespace， 而是**和宿主机共用一个Network Namespace**。容器将不会虚拟出自己的网卡而是使用宿主机的IP和端口。

![image-20220721024433315](img.assets\image-20220721024433315.png)



**实例：**

```
docker run -d  --network host --name tomcat tomcat:9.0.55
```

不推荐使用`-p`指定端口启动

```
docker run -d -p 8083:8080 --network host --name tomcat tomcat:9.0.55
```

![image-20220721024925251](img.assets\image-20220721024925251.png)

警告说明:

docker启动时指定--network=host或-net=host，如果还指定了-p映射端口，这时就会有此警告，并且通过-p设置的参数将不会起到任何作用，**端口号会以主机端口号为主，重复时则递增**。

解决的办法就是使用docker的其他网络模式，例如--network=bridge，这样就可以解决问题，或者直接无视



1.查看宿主机网络配置

![image-20220721025526026](img.assets\image-20220721025526026.png)

2.查看tomcat容器网络配置,**没有ip和网关的配置，和主机使用一套网络配置**

![image-20220721025914241](img.assets\image-20220721025914241.png)



### 3.3.4 none模式

**禁用网络功能，只有lo标识**

在none模式下，并不为Docker容器进行任何网络配置。 也就是说，这个Docker容器没有网卡、IP、路由等信息，只有一个lo，需要我们自己为Docker容器添加网卡、配置IP等。



**实例：**

```
docker run -d --network none --name tomcat tomcat:9.0.55
```

![image-20220721030637057](img.assets\image-20220721030637057.png)

进入容器实例查看：

![image-20220721030727846](img.assets\image-20220721030727846.png)



### 3.3.5 container模式

新建的容器和**已经存在的一个容器共享一个网络ip配置**而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。

![image-20220721030950076](img.assets\image-20220721030950076.png)

使用命令:`--network container:NAME`或者容器ID指定



**错误实例：**

```
docker run -d -p 8080:8080 --name tomcat80 tomcat:9.0.55
docker run -d -p 8081:8080 --network container:tomcat80 --name tomcat81 tomcat:9.0.55
```

![image-20220721031502121](img.assets\image-20220721031502121.png)

**两个容器实例公用一个网络ip配置，导致两个容器上的tomcat都使用同一个8080端口，发生冲突。相当于在一台机器上启动两个8080端口的tomcat**



**正确实例:**

Alpine Linux 是一款独立的、非商业的通用 Linux 发行版，专为追求安全性、简单性和资源效率的用户而设计。因为他小，简单，安全而著称，所以作为基础镜像是非常好的一个选择，麻雀虽小但五脏俱全，镜像非常小巧，不到 6M的大小，所以特别适合容器打包。

![image-20220721032034143](img.assets\image-20220721032034143.png)

启动alpine

```
docker run -it --name alpine1 alpine /bin/sh
docker run -it --network container:alpine1 --name alpine2 alpine /bin/sh
```

![image-20220721032247882](img.assets\image-20220721032247882.png)

查看alpine的网络配置:

alpine1:

![image-20220721032549620](img.assets\image-20220721032549620.png)

alpine2:**和alpine1网络配置完全相同**

![image-20220721032630419](img.assets\image-20220721032630419.png)

若关闭alpine1，查看alpine2:**alpine2只能在本地访问,再次启动alpine1，不发生变化**

![image-20220721032922835](img.assets\image-20220721032922835.png)



### 3.3.6 自定义网络模式

容器IP变动时候可以**通过服务名直接网络通信**而不受到影响

**自定义网络本身就维护好了主机名和ip的对应关系（ip和域名都能通）**



1.新建自定义网络，默认使用bridge模式

```
docker network create xh_network
```

![image-20220721033954115](img.assets\image-20220721033954115.png)

2.新建容器加入自定义网络

```
docker run -d -p 8080:8080 --network xh_network --name tomcat80 tomcat:9.0.55
docker run -d -p 8081:8080 --network xh_network --name tomcat81 tomcat:9.0.55
```

![image-20220721034219957](img.assets\image-20220721034219957.png)



3.进入容器中**使用服务名进行网络通信**

![image-20220721034520553](img.assets\image-20220721034520553.png)



# 四.Docker-compose容器编排

Compose 是 Docker 公司推出的一个工具软件，可以管理多个 Docker 容器组成一个应用。你需要定义一个 **YAML** 格式的配置文件**docker-compose.yml**，写**好多个容器之间的调用关系**。然后，只要一个命令，就能同时启动/关闭这些容器



## 4.1 容器编排的作用


​		docker建议我们**每一个容器中只运行一个服务**,因为docker容器本身占用资源极少,所以最好是将每个服务单独的分割开来,docker官方给我们提供了docker-compose多服务部署的工具,避免为每个服务单独写Dockerfile然后在构建镜像。Compose允许用户通过一个单独的docker-compose.yml模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。可以很容易地用一个配置文件定义一个多容器的应用，然后使用一条指令安装这个应用的所有依赖，完成构建。**Docker-Compose 解决了容器与容器之间如何管理编排的问题。**



## 4.2 安装

[官网](https://docs.docker.com/compose/compose-file/compose-file-v3/)

[官网下载](https://docs.docker.com/compose/install/)

注意版本要对应

![image-20220721184856932](img.assets\image-20220721184856932.png)



安装步骤:

```shell
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

docker-compose --version
```

![image-20220721195015530](img.assets\image-20220721195015530.png)

卸载步骤:

```
rm /usr/local/bin/docker-compose
```



## 4.3 Compose核心概念

一文件：**docker-compose.yml文件**

两要素:

​	1.服务(service):· 一个个应用容器实例，比如订单微服务、库存微服务、mysql容器、nginx容器或者redis容器

​    2.工程(project):· 由一组关联的应用容器组成的一个**完整业务单元**，在 docker-compose.yml 文件中定义。



## 4.4 Compose使用的三个步骤

1. 编写Dockerfile定义各个微服务应用并构建出对应的镜像文件

2. 使用 docker-compose.yml 定义一个完整业务单元，安排好整体应用中的各个容器服务。

3. 最后，执行`docker-compose up`命令 来启动并运行整个应用程序，完成一键部署上线



## 4.5 Compose常用命令 

| 命令                                 | 说明                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| docker-compose -h                    | 查看帮助                                                     |
| docker-compose up                    | 启动所有docker-compose服务                                   |
| **docker-compose up -d**             | **启动所有docker-compose服务并后台运行**                     |
| **docker-compose down**              | 停止并删除容器、网络、卷、镜像。                             |
| docker-compose exec  yml里面的服务id | 进入容器实例内部 docker-compose exec **docker-compose.yml文件中写的服务id**  /bin/bash |
| docker-compose ps                    | 展示当前docker-compose编排过的运行的所有容器                 |
| docker-compose top                   | 展示当前docker-compose编排过的容器进程                       |
| docker-compose logs  yml里面的服务id | 查看容器输出日志                                             |
| **docker-compose config**            | **检查配置**                                                 |
| **docker-compose config -q**         | **检查配置，有问题才有输出**                                 |
| docker-compose restart               | 重启服务                                                     |
| docker-compose start                 | 启动服务                                                     |
| docker-compose stop                  | 停止服务                                                     |



## 4.6  Compose编排微服务

### 4.6.1 不使用Compose

1.启动单独的mysql容器，并且创建相关表和数据

```shell
docker run -d -p 3306:3306 --privileged=true -v /root/mysql/log:/var/log/mysql -v /root/mysql/data:/var/lib/mysql -v /root/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=root --name mysql mysql:5.7.30
```

2.启动redis容器

```shell
docker run -p 6379:6379 --name redis --privileged=true -v /root/redis/redis.conf:/etc/redis/redis.conf -v /root/redis/data:/data -d redis:6.2.6 redis-server /etc/redis/redis.conf
```

3.启动微服务

```shell
docker run -d -p 8888:8888 --name docker_boot docker_boot:1.0.0
```

![image-20220723012805571](img.assets\image-20220723012805571.png)

使用swagger测试 http://192.168.153.140:8888/swagger-ui/index.htm 成功



不使用Compose的问题：

1.多个run命令......

2.容器间的启停或宕机，有可能导致IP地址对应的容器实例变化，映射出错， 要么生产IP写死(可以但是不推荐)，要么通过服务调用



### 4.6.2  使用Compose

1.编写`docker-compose.yml文件`

```
version: "3" #docker-compose版本

 
services:

  docker_boot: #服务名称:自定义

    image: docker_boot:1.0.0 #要启动的服务镜像
 
    container_name: docker_boot  #镜像名称,若不指定则使用文件名_服务名_数字来命名

    ports:  #暴露端口号

      - "8888:8888" 

    volumes: #容器数据卷

      - /root/app/docker_boot:/data

    networks: #网络,使用自定义网络后可以不用写死ip地址，使用服务名调用

      - xh_net 

    depends_on: #该服务依赖于redis和mysql镜像（redis和mysql是服务名）

      - redis

      - mysql


  redis:

    image: redis:6.2.6
    
    ports:

      - "6379:6379"

    volumes:

      - /root/app/redis/redis.conf:/etc/redis/redis.conf

      - /root/app/redis/data:/data

    networks: 

      - xh_net 

    command: redis-server /etc/redis/redis.conf

 

  mysql:

    image: mysql:5.7.30

    environment:

      MYSQL_ROOT_PASSWORD: 'root'

      MYSQL_ALLOW_EMPTY_PASSWORD: 'no'

      MYSQL_DATABASE: 'db2022'

      MYSQL_USER: 'root'

      MYSQL_PASSWORD: 'root'

    ports:

       - "3306:3306"

    volumes:

       - /root/app/mysql/db:/var/lib/mysql

       - /root/app/mysql/conf/my.cnf:/etc/my.cnf

       - /root/app/mysql/init:/docker-entrypoint-initdb.d

    networks:

      - xh_net

    command: --default-authentication-plugin=mysql_native_password #解决外部无法访问

 

networks:  #自定义网络

   xh_net: 
```

2.检查docker-compose.yml文件是否出错,**没有任何提示则说明正确**

```shell
docker-compose config -q
```

![image-20220723022050293](img.assets\image-20220723022050293.png)

3.使用自定义网络后可以使用服务名来代替ip

![image-20220723020625956](img.assets\image-20220723020625956.png)

4.使用`docker-compose up -d`来启动

![image-20220723022634165](img.assets\image-20220723022634165.png)

5.在mysql中创建表和相关数据

6.使用swagger进行测试http://192.168.153.140:8888/swagger-ui/index.htm 成功

7.停止compose

```
docker-compose down
```

![image-20220723023119470](img.assets\image-20220723023119470.png)





# 五.Portainer

Portainer 是一款轻量级的应用，它提供了图形化界面，用于方便地管理Docker环境，包括单机环境和集群环境。

[官网](https://www.portainer.io/)

[下载](https://docs.portainer.io/v/ce-2.9/start/install/server/docker/linux)



## 5.1 Portainer安装

docker命令安装 linux环境下的Portainer `--restart=always`:docker重启，Portainer也重启

```shell
docker run -d -p 8000:8000 -p 9000:9000 --name portainer --restart=always  -v /var/run/docker.sock:/var/run/docker.sock  -v portainer_data:/data   portainer/portainer-ce
```

![image-20220723024409329](img.assets\image-20220723024409329.png)



1.访问Portainer`ip:9000`，创建用户名和密码，密码要8位以上

![image-20220723024507549](img.assets\image-20220723024507549.png)

2.登录

![image-20220723024723263](img.assets\image-20220723024723263.png)

![image-20220723024743209](img.assets\image-20220723024743209.png)

stacks:容器编排数量

containers:容器数量

images:镜像的数量

Volumes:容器卷的数量

network:docke网络数量

底层命令：`docker system df`



# 六.CIG容器监控系统

原生命令:`docker stats`

docker stats统计结果只能是当前宿主机的全部容器，数据资料是实时的，没有地方存储、没有健康指标过线预警等功能



**CLG:CAdvisor监控收集+InfluxDB存储数据+Granfana展示图表**

![image-20220723031158527](img.assets\image-20220723031158527.png)

## 6.1 CAdvisor

​		CAdvisor是一个容器资源监控工具包括容器的内存,CPU,网络IO,磁盘IO等监控,同时提供了一个WEB页面用于查看容器的实时运行状态。CAdvisor默认存储2分钟的数据,而且只是针对单物理机。不过，CAdvisor提供了很多数据集成接口,支持InfluxDB,Redis,Kafka,Elasticsearch等集成,可以加上对应配置将监控数据发往这些数据库存储起来。



CAdvisor功能主要有两点:

​	1.展示Host和容器两个层次的监控数据。

​	2.展示历史变化数据。



## 6.2 InfluxDB

InfluxDB是用Go语言编写的一个开源分布式时序、事件和指标数据库,无需外部依赖。

CAdvisor默认只在本机保存最近2分钟的数据，为了持久化存储数据和统一收集展示监控数据，需要将数据存储到InfluxDB中。InfluxDB是一个时序数据库,专门用于存储时序相关数据，很适合存储CAdvisor的数据。而且，CAdvisor本身已经提供了InfluxDB的集成方法，丰启动容器时指定配置即可。



lnfluxDB主要功能:

​	1.基于时间序列,支持与时间有关的相关函数(如最大、最小、求和等);

​	2.可度量性:你可以实时对大量数据进行计算;

​	3.基于事件:它支持任意的事件数据;



## 6.3 Granfana

Grafana是一个开源的数据监控分析可视化平台,支持多种数据源配置(支持的数据源包括InfluxDB,MySQL,Elasticsearch,OpenTSDB,Graphite等)和丰富的插件及模板功能,支持图表权限控制和报警。



Grafan主要特性:

1.灵活丰富的图形化选项。

2.可以混合多种风格。

3.支持白天和夜间模式

4.多个数据源



## 6.4 使用compose搭建CIG

新建目录clg

```shell
mkdir clg
```

创建`docker-compose.yml`文件

```
version: '3.1'

volumes:

  grafana_data: {}

services:

 influxdb:

  image: tutum/influxdb:0.9

  restart: always

  environment:

    - PRE_CREATE_DB=cadvisor #初始数据库设置

  ports:

    - "8083:8083"

    - "8086:8086"

  volumes:

    - ./data/influxdb:/data

 

 cadvisor:

  image: google/cadvisor

  links:

    - influxdb:influxsrv

  command: 
  
  	- storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=influxsrv:8086

  restart: always

  ports:

    - "8080:8080"

  volumes:

    - /:/rootfs:ro

    - /var/run:/var/run:rw

    - /sys:/sys:ro

    - /var/lib/docker/:/var/lib/docker:ro

 

 grafana:

  user: "104"

  image: grafana/grafana

  user: "104"

  restart: always

  links:

    - influxdb:influxsrv

  ports:

    - "3000:3000"

  volumes:

    - grafana_data:/var/lib/grafana

  environment:

    - HTTP_USER=admin

    - HTTP_PASS=admin

    - INFLUXDB_HOST=influxsrv

    - INFLUXDB_PORT=8086

    - INFLUXDB_NAME=cadvisor

    - INFLUXDB_USER=root

    - INFLUXDB_PASS=root
```

检查是否有语法错误`docker-compose config -q`

启动`docker-compose up -d`

![image-20220723033515088](img.assets\image-20220723033515088.png)



### 6.4.1 查看CAdvisor

`http://ip:8080`

![image-20220723034143590](img.assets\image-20220723034143590.png)



### 6.4.2 查看InfluxDB

`http://ip:8083`

![image-20220723034241741](img.assets\image-20220723034241741.png)



### 6.4.3 查看Granfana

`http://ip:3000`  

![image-20220723034337716](img.assets\image-20220723034337716.png)

默认的用户名和密码为admin

![image-20220723034447520](img.assets\image-20220723034447520.png)



## 6.5 CLG配置panel

1.为Granfana配置数据源

![image-20220723034629887](img.assets\image-20220723034629887.png)

2.数据源选择InfluxDB

![image-20220723034728657](img.assets\image-20220723034728657.png)

3.InfluxDB配置

​	3.1 尽量使用服务名去调用InfluxDB

![image-20220723034944542](img.assets\image-20220723034944542.png)

​			3.2 数据库设置

![image-20220723035347603](img.assets\image-20220723035347603.png)

配置成功

![image-20220723035422207](img.assets\image-20220723035422207.png)

4.创建新的观察页面

​	4.1

![image-20220723035601249](img.assets\image-20220723035601249.png)

 	4.2

![image-20220723035655042](img.assets\image-20220723035655042.png)

​	4.3

![image-20220723035820936](img.assets\image-20220723035820936.png)

选择图形化展示效果图

![image-20220723035912605](img.assets\image-20220723035912605.png)

4.4 填写相关描述信息，点击save

![image-20220723040144088](img.assets\image-20220723040144088.png)



## 6.6 CLG配置监控业务规则

1.进入配置

![image-20220723040345429](img.assets\image-20220723040345429.png)

2.

![image-20220723042050461](img.assets\image-20220723042050461.png)
