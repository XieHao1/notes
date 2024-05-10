

[TOC]

# Docker

Docker 是一个开源的应用容器引擎，基于 Go 语言并遵从 Apache2.0 协议开源。

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

Docker的主要目标是“Build，Ship and Run Any App,Anywhere”，也就是通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户的APP（可以是一个WEB应用或数据库应用等等）及其运行环境能够做到**“一次镜像，处处运行**”



# 1.Docker的应用场景

- Web 应用的自动化打包和发布。
- 自动化测试和持续集成、发布。
- 在服务型环境中部署和调整数据库或其他的后台应用。
- 从头编译或者扩展现有的 OpenShift 或 Cloud Foundry 平台来搭建自己的 PaaS 环境。



# 2.容器和虚拟机



## 2.1 虚拟机

虚拟机（virtual machine）就是带环境安装的一种解决方案。

它可以在一种操作系统里面运行另一种操作系统，比如在Windows10系统里面运行Linux系统CentOS7。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。这类虚拟机完美的运行了另一套系统，能够使应用程序，操作系统和硬件三者之间的逻辑不变。 



**虚拟机的缺点：**

1.资源占用多 

2.冗余步骤多        

3.启动慢



## 2.2  容器虚拟化

Linux容器(Linux Containers，缩写为 LXC)

Linux容器是与系统其他部分隔离开的一系列进程，**从另一个镜像运行，并由该镜像提供支持进程所需的全部文件**。容器提供的镜像包含了应用的所有依赖项，因而在从开发到测试再到生产的整个过程中，它都具有可移植性和一致性。

 

Linux 容器不是模拟一个完整的操作系统而是对进程进行隔离。有了容器，就可以将软件运行所需的所有资源打包到一个隔离的容器中。容器与虚拟机不同，不需要捆绑一整套操作系统，只需要软件工作所需的库资源和设置。系统因此而变得高效轻量并保证部署在任何环境中的软件都能始终如一地运行。

![image-20220616155834127](img.assets\image-20220616155834127.png)



Docker容器是在**操作系统层面上实现虚拟化**，直接复用本地主机的操作系统，而传统虚拟机则是在硬件层面实现虚拟化。与传统的虚拟机相比，Docker优势体现为启动速度快、占用体积小。



## 2.3 区别

1.传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；

2.容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

3.每个容器之间互相隔离，每个容器有自己的文件系统 ，容器之间进程不会相互影响，能区分计算资源。



## 2.4 Docker比虚拟机快

**(1)docker有着比虚拟机更少的抽象层**

  由于docker不需要Hypervisor(虚拟机)实现硬件资源虚拟化,运行在docker容器上的程序**直接使用的都是实际物理机的硬件资源**。因此在CPU、内存利用率上docker将会在效率上有明显优势。

**(2)docker利用的是宿主机的内核,而不需要加载操作系统OS内核**

  当新建一个容器时,docker不需要和虚拟机一样重新加载一个操作系统内核。进而避免引寻、加载操作系统内核返回等比较费时费资源的过程,当新建一个虚拟机时,虚拟机软件需要加载OS,返回新建过程是分钟级别的。而docker由于直接利用宿主机的操作系统,则省略了返回过程,因此新建一个docker容器只需要几秒钟。

![image-20220616201516090](img.assets\image-20220616201516090.png)



# 3.Docker 的优点

Docker 是一个用于开发，交付和运行应用程序的开放平台。Docker 使您能够将应用程序与基础架构分开，从而可以快速交付软件。借助 Docker，您可以与管理应用程序相同的方式来管理基础架构。通过利用 Docker 的方法来快速交付，测试和部署代码，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。



## 3.1 快速，一致地交付应用程序

Docker 允许开发人员使用您提供的应用程序或服务的本地容器在标准化环境中工作，从而简化了开发的生命周期。



## 3.2 响应式部署和扩展

Docker 是基于容器的平台，允许高度可移植的工作负载。Docker 容器可以在开发人员的本机上，数据中心的物理或虚拟机上，云服务上或混合环境中运行。

Docker 的可移植性和轻量级的特性，还可以使您轻松地完成动态管理的工作负担，并根据业务需求指示，实时扩展或拆除应用程序和服务



## 3.3 在同一硬件上运行更多工作负载

Docker 轻巧快速。它为基于虚拟机管理程序的虚拟机提供了可行、经济、高效的替代方案，因此您可以利用更多的计算能力来实现业务目标。Docker 非常适合于高密度环境以及中小型部署，而您可以用更少的资源做更多的事情。



# 4.Docker架构

Docker 包括三个基本概念:

- **镜像（Image）**：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。

- **容器（Container）**：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。每个容器都是相互隔离的、保证安全的平台

  **可以把容器看做是一个简易版的 Linux 环境**（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

- **仓库（Repository）**：仓库可看成一个代码控制中心，用来保存镜像。

  类似于

  Maven仓库，存放各种jar包的地方；

  github仓库，存放各种git项目的地方；

  Docker公司提供的官方registry被称为Docker Hub，存放各种镜像模板的地方。

   

  仓库分为公开仓库（Public）和私有仓库（Private）两种形式。

  最大的公开仓库是 [Docker Hub](https://hub.docker.com/)，

  存放了数量庞大的镜像供用户下载。国内的公开仓库包括阿里云 、网易云等

  

Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。

Docker 容器通过 Docker 镜像来创建。

**容器与镜像的关系类似于面向对象编程中的对象与类。**

| Docker | 面向对象 |
| :----- | :------- |
| 容器   | 对象     |
| 镜像   | 类       |

![img](img.assets\576507-docker1.png)

| 概念                   | 说明                                                         |
| :--------------------- | :----------------------------------------------------------- |
| Docker 镜像(Images)    | Docker 镜像是用于创建 Docker 容器的模板，比如 Ubuntu 系统。  |
| Docker 容器(Container) | 容器是独立运行的一个或一组应用，是镜像运行时的实体。         |
| Docker 客户端(Client)  | Docker 客户端通过命令行或者其他工具使用 Docker SDK (https://docs.docker.com/develop/sdk/) 与 Docker 的守护进程通信。 |
| Docker 主机(Host)      | 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。       |
| Docker Registry        | Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。Docker Hub([https://hub.docker.com](https://hub.docker.com/)) 提供了庞大的镜像集合供使用。一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 **<仓库名>:<标签>** 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 **latest** 作为默认标签。 |
| Docker Machine         | Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。 |



## 4.1 Docker工作原理

Docker是一个Client-Server结构的系统，Docker守护进程运行在主机上， 然后通过Socket连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。 容器，是一个运行时环境，就是我们前面说到的集装箱。类似MYSQL

![image-20220616171546899](img.assets\image-20220616171546899.png)



## 4.2 Docker运行流程

Docker 是一个 C/S 模式的架构，后端是一个松耦合架构，众多模块各司其职。 

Docker运行的基本流程为:

1 用户是使用Docker Client与 Docker Daemon建立通信，并发送请求给后者。

2 Docker Daemon作为Docker架构中的主体部分，首先提供Docker Server的功能使其可以接受Docker Client的请求。

3 Docker Engine执行Docker 内部的一系列工作，每一项工作都是以一个Job的形式的存在。

4 Job的运行过程中，当需要容器镜像时，则从Docker Registry中下载镜像，并通过镜像管理驱动Graph driver将下载镜像以Graph的形式存储。

5 当需要为Docker创建网络环境时，通过网络管理驱动Network driver创建并配置Docker容器网络环境。

6 当需要限制Docker容器运行资源或执行用户指令等操作时，则通过Exec driver来完成。

7 Libcontainer是一项独立的容器管理包，Network driverl以及Exec driver都是通过Libcontainer来实现具体对容器进行的操作。

![image-20220616172045072](img.assets\image-20220616172045072.png)

# 5.Docker安装

[docker官网](http://www.docker.com)

[Docker Hub官网](https://hub.docker.com/)

[CentOS安装官方教程](https://docs.docker.com/engine/install/centos/)

![image-20220616161741828](img.assets\image-20220616161741828.png)



## 5.1 安装步骤

1.确定为CentOS7及以上版本

```
cat /etc/redhat-release
```

2.卸载旧版本

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

3.yum安装gcc相关

```shell
yum -y install gcc
yum -y install gcc-c++
```

4.安装需要的软件包

```shell
yum install -y yum-utils
```

5.设置stable镜像仓库

使用官网推荐安装缓慢

```shell
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

**推荐使用阿里云仓库**

```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

清华大学源

```shell
yum-config-manager \
    --add-repo \
    https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

6.更新yum软件包索引

```shell
yum makecache fast
```

若出错则去掉 fast

7.安装DOCKER CE

```
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

8.启动docker

```
systemctl start docker
```

9.测试

```
docker version
```

```
docker run hello-world
```

10.卸载 

```
systemctl stop docker
```

```
yum remove docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

```
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```



## 5.2 阿里云镜像加速

国内从 DockerHub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。Docker 官方和国内很多云服务商都提供了国内加速器服务

- 科大镜像：**https://docker.mirrors.ustc.edu.cn/**
- 网易：**https://hub-mirror.c.163.com/**
- 阿里云：**https://<你的ID>.mirror.aliyuncs.com**
- 七牛云加速器：**https://reg-mirror.qiniu.com**

阿里云镜像：

![image-20220616200822324](img.assets\image-20220616200822324.png)



粘贴配置命令：

![image-20220616200932245](img.assets\image-20220616200932245.png)

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://5zwwn9q9.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## 5.3 run命令

![image-20220616201350292](img.assets\image-20220616201350292.png)



# 6.Docker常用命令

## 6.1 帮助启动类命令

1.启动docker：`systemctl start docker`

2.停止docker：`systemctl stop docker`

3.重启docker: `systemctl restart docker`

4.查看docker状态: `systemctl status docker`

5.开机 启动: `systemctl enable docker`

6.查看docker概要信息: `docker info`

7.查看docker总体帮助文档: `docker --help`

8.查看docker命令帮助文档: `docker 具体命令 --help`



## 6.2 镜像命令

### 6.2.1 docker images

**列出本地主机上的镜像**

![image-20220629211331028](img.assets\image-20220629211331028.png)

REPOSITORY：表示镜像的仓库源

TAG：镜像的标签版本号

IMAGE ID：镜像ID

CREATED：镜像创建时间

SIZE：镜像大小

**同一仓库源可以有多个 TAG版本**，代表这个仓库源的不同个版本，我们使用 **REPOSITORY:TAG 来定义不同的镜像**。

如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用`ubuntu:latest` 镜像



**参数说明:**

​	-a: 列出本地所有的镜像（含历史印象层）

​	-q:只显示镜像ID



### 6.2.2 docker search 镜像名

我们可以从 Docker Hub 网站来搜索镜像，Docker Hub 网址为： **https://hub.docker.com/**

我们也可以使用 `docker search` 命令来**搜索镜像**

![image-20220629212213376](img.assets\image-20220629212213376.png)

**NAME:** 镜像仓库源的名称

**DESCRIPTION:** 镜像的描述

**stars:** 类似 Github 里面的 star，表示点赞、喜欢的意思。

**OFFICIAL:** 是否 docker 官方发布

**AUTOMATED:** 自动构建。



**参数说明:**

​	--limit: 只列出N个镜像，默认25个

```
 docker search --limit 5 redis
```



### 6.2.3  docker pull 镜像名

命令 `docker pull` **下载镜像**

```
dokcer pull 镜像名字[:TAG]  //[] 可选，指定版本号
docker pull 镜像名字 //默认为最新版  等价于 docker pull 镜像名字:latest
```

![image-20220629213243539](img.assets\image-20220629213243539.png)



### 6.2.4 docker system df 

**查看镜像/容器/数据卷所占的空间**

![image-20220629213838450](img.assets\image-20220629213838450.png)



### 6.2.5 docker rmi 镜像ID

**删除镜像**

![image-20220629214400515](img.assets\image-20220629214400515.png)



1.删除单个镜像: `docker rmi 镜像ID[:TAG] `

2.删除多个镜像: `dokcer rmi 镜像ID1[:TAG] 镜像ID2[:TAG]`

3.全部删除:`docker rmi $(docker images -qa)`



**参数说明:**

​	-f : 强制删除



### 6.2.6 docker的虚悬镜像

仓库名，标签 都是<none>的镜像，俗称虚悬镜像 dangling image

**没有什么作用，一般将其删除**

![](img.assets\image-20220630114900862.png)



## 6.3 容器命令

**有镜像才能创建容器（下载一个CentOS或者ubuntu镜像）**，推荐使用ubuntu，体积小

```
docekr pull ubuntu
```

![image-20220629215628727](img.assets\image-20220629215628727.png)

### 6.3.1 新建+启动容器

`docker run [optiins] image [command][arg...]`



 **OPTIONS说明（常用）：**有些是一个减号，有些是两个减号

​	--name="容器新名字" :为容器指定一个名称(**实例名称**)；



​	-d: 后台运行容器并返回容器ID，也即启动守护式容器(后台运行)； **--后台守护式启动（后台运行程序启动）**

​	-i：以交互模式运行容器，通常与 -t 同时使用；**(交互式操作)**  **-- 前台交互式启动(一般操作系统使用)**

​	-t：为容器重新分配一个伪输入终端，通常与 -i 同时使用；也即启动交互式容器(前台有伪终端，等待交互)；**(终端)**

​	#使用镜像ubuntu:latest以交互模式启动一个容器,在容器内执行/bin/bash命令

​	#/bin/bash：放在镜像名后的是命令，这里我们希望有个交互式 **Shell**，因此用的是 /bin/bash,要退出终端，直接输入 exit:

![image-20220629221428818](img.assets\image-20220629221428818.png)



​	-P: 随机端口映射，大写P

​	**-p: 指定端口映射，小写p(一般使用-p)**

![image-20220629220757280](img.assets\image-20220629220757280.png)



### 6.3.2 查看正在运行的容器实例

`docekr -ps [OPTINOS]`



OPTIONS说明（常用）：

-a :列出当前所有正在运行的容器+历史上运行过的

-l :显示最近创建的容器。

-n 数字：显示最近n个创建的容器。

-q :静默模式，只显示容器编号。

![image-20220629222808339](img.assets\image-20220629222808339.png)



### 6.3.3 退出容器

​	1.`exit`:run进去容器 ，exit退出，**容器停止**

​	2.`ctrl+p+q`:run进去容器，ctrl+p+q退出，**容器不停止**,可以使用`docker exec -it 容器名 `id /bin/bash`进入伪终端

![image-20220629223908455](img.assets\image-20220629223908455.png)

![image-20220629223930359](img.assets\image-20220629223930359.png)



### 6.3.4 启动已经停止的容器

`docker start 容器的id或者容器名`

![image-20220629224144287](img.assets\image-20220629224144287.png)



### 6.3.5 重启容器

`docker restart 容器的id或者容器名`

![image-20220629224350071](img.assets\image-20220629224350071.png)



### 6.3.6 停止容器

`docker stop 容器的id或者容器名`

![image-20220629224506329](img.assets\image-20220629224506329.png)



**强制停止容器**:`docker kill 容器的id或者容器名 `



### 6.3.7 删除已经停止的容器

1.单个删除停止的容器:`docekr rm 容器id或者容器名 `,可以使用 `-f` 进行强制删除

![image-20220629224833123](img.assets\image-20220629224833123.png)

2. 一次性删除多个容器实例

   ​	`docker rm -f $(docker ps -a -q)`

​		   `docker ps -a -q | xargs docker rm`



### 6.3.8 !启动守护式容器(后台服务器)!

在大部分的情况下，我们希望docker的服务实在后台运行的，可以通过`-d` 指定容器的后台运行模式

`docker run -d 镜像名`



相关问题说明:

启动ubuntu容器，然后docker ps 进行查看, 会发现容器已经退出：

![image-20220629231353804](img.assets\image-20220629231353804.png)

**Docker容器后台运行,就必须有一个前台进程。**容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就是会自动退出的。



使用前后台方式分别启动redis

​	1.前台交互式启动`docker run -it redis:6.0.8`,使用`ctrl+c`强制退出

![image-20220629232006621](img.assets\image-20220629232006621.png)

​	2.后台守护式启动 `docker run -d redis:6.0.8`

![image-20220629232150627](img.assets\image-20220629232150627.png)



### 6.3.9 !查看容器日志!

`docker logs 容器id`

![image-20220629232531577](img.assets\image-20220629232531577.png)



### 6.3.10 !查看容器内运行的进程!

`docker top 容器id`

![image-20220629232918427](img.assets\image-20220629232918427.png)



### 6.3.11 !查看容器内部细节!

`docker inspect 容器id`

![image-20220629233053762](img.assets\image-20220629233053762.png)



### 6.3.12 !进入运行的容器!

`docker exec -it 容器id /bin/bash` --- 在容器中打开新的终端。并且可以启动新的进程，**用exit退出，不会导致容器停止(推荐使用这个)**

![image-20220630111909472](img.assets\image-20220630111909472.png)

![image-20220630112548094](img.assets\image-20220630112548094.png)



`docker attach 容器id` ---- 直接进入容器的启动命令的终端，不会启动新的进程，用exit退出，会导致容器停止

一般用`-d`后台启动的程序，再用exec进入对应的容器实例



### 6.3.13 !容器文件拷贝到主机!

`docker cp 容器id:容器内路径 目的主机路径`

![image-20220630113839563](img.assets\image-20220630113839563.png)



### 6.3.14 !导入和导出容器!

`export` 导出容器的内容留作为一个tar归档文件[对应import命令]

`docker export 容器id > 文件名.tar`

![image-20220630114547958](img.assets\image-20220630114547958.png)



`import` 从tar包中的内容创建一个新的文件系统再导入为镜像[对应export]

`cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号 `

![image-20220630115254263](img.assets\image-20220630115254263.png)

**容器的导入和导出是对整个容器进行备份处理**



## 6.4 命令总结

![image-20220630115556133](img.assets\image-20220630115556133.png)



|  命令   |                             说明                             |
| :-----: | :----------------------------------------------------------: |
| attach  |            当前 shell 下 attach 连接指定运行镜像             |
|  build  |                   通过 Dockerfile 定制镜像                   |
| commit  |                    提交当前容器为新的镜像                    |
|   cp    |            从容器中拷贝指定文件或者目录到宿主机中            |
| create  |            创建一个新的容器，同 run，但不启动容器            |
|  diff   |                     查看 docker 容器变化                     |
| events  |                   在已存在的容器上运行命令                   |
| export  |     导出容器的内容流作为一个 tar 归档文件[对应 import ]      |
| import  |     从tar包中的内容创建一个新的文件系统映像[对应export]      |
| history |                     展示一个镜像形成历史                     |
| images  |                       列出系统当前镜像                       |
|  info   |                       显示系统相关信息                       |
| inspect |                       查看容器详细信息                       |
|  kill   |                    kill 指定 docker 容器                     |
|  load   |            从一个 tar 包中加载一个镜像[对应 save]            |
|  save   |             保存一个镜像为一个 tar 包[对应 load]             |
|  login  |               注册或者登陆一个 docker 源服务器               |
| logout  |                 从当前 Docker registry 退出                  |
|  logs   |                     输出当前容器日志信息                     |
|  port   |               查看映射端口对应的容器内部源端口               |
|  pause  |                           暂停容器                           |
|   ps    |                         列出容器列表                         |
|  pull   |          从docker镜像源服务器拉取指定镜像或者库镜像          |
|  push   |            推送指定镜像或者库镜像至docker源服务器            |
| restart |                        重启运行的容器                        |
|   rm    |                     移除一个或者多个容器                     |
|   rmi   | 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或 -f 强制删除] |
|   run   |                创建一个新的容器并运行一个命令                |
| search  |                   在 docker hub 中搜索镜像                   |
|  start  |                           启动容器                           |
|  stop   |                           停止容器                           |
|   tag   |                       给源中镜像打标签                       |
|   top   |                   查看容器中运行的进程信息                   |
| unpause |                         取消暂停容器                         |
| version |                      查看 docker 版本号                      |
|  wait   |                  截取容器停止时的退出状态值                  |



# 7.docker镜像

**镜像**是一种轻量级、可执行的独立软件包，它包含运行某个软件所需的所有内容，我们把应用程序和配置依赖打包好形成一个可交付的运行环境(包括代码、运行时需要的库、环境变量和配置文件等)，这个打包好的运行环境就是image镜像文件。只有通过这个镜像文件才能生成Docker容器实例(类似Java中new出来一个对象)。



**Docker镜像层都是只读的，容器层是可写的。**

当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。

所有对容器的改动 - 无论添加、删除、还是修改文件都只会发生在容器层中。只有容器层是可写的，容器层下面的所有镜像层都是只读的。

![image-20220702080025530](img.assets\image-20220702080025530.png)



**Docker中的镜像分层，支持通过扩展现有镜像，创建新的镜像**。类似Java继承于一个Base基础类，自己再按需扩展。

新镜像是从 base 镜像一层一层叠加生成的。每安装一个软件，就在现有镜像的基础上增加一层



## 7.1 镜像的分层

通过`pull` 命令 ，可以看到镜像是分层下载的

![image-20220630120728046](img.assets\image-20220630120728046.png)



## 7.2 UnionFS-联合文件系统

UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它**支持对文件系统的修改作为一次提交来一层层的叠加，**同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。UnionFS 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。



**特性**：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录



## 7.3 Docker 镜像的加载原理

  docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。



bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，**在Docker镜像的最底层是引导文件系统bootfs**。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。



rootfs (root file system) ，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。 



对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供 rootfs 就行了。由此可见对于不同的linux发行版, bootfs基本是一致的, rootfs会有差别, 因此不同的发行版可以公用bootfs。



## 7.4 Docker镜像分层的好处

**镜像分层最大的一个好处就是共享资源，方便复制迁移，就是为了复用。**

比如说有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像；同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。



## 7.5 Docker的commit命令

`docker commit` 提交容器副本使之称为一个新的镜像

`docker commit -m="提交的描述信息 -a="作者" 容器ID 要创建的目标镜像名:[标签名]`



在原本的ubuntu镜像中安装vim命令并且提交为新的镜像

安装vim: 

```
agt-get update
agt-get -y install vim
```

提交为新的镜像

![image-20220702081934050](img.assets\image-20220702081934050.png)





# 8.本地镜像发布上传

## 8.1 发布的阿里云

![image-20220702082714056](img.assets\image-20220702082714056.png)

### 8.1.1 创建阿里云镜像仓库

![image-20220702083230791](img.assets\image-20220702083230791.png)

创建命名空间和仓库名称

![image-20220702083508891](img.assets\image-20220702083508891.png)

![image-20220702083721559](img.assets\image-20220702083721559.png)



### 8.1.2 将镜像推送到阿里云

仓库建立完毕后会生成相关命令

![image-20220702083926044](img.assets\image-20220702083926044.png)

```shell
$ docker login --username=xh169306265 registry.cn-hangzhou.aliyuncs.com
$ docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/xh-images/my-registry:[镜像版本号]
$ docker push registry.cn-hangzhou.aliyuncs.com/xh-images/my-registry:[镜像版本号]
```

![image-20220702084532455](img.assets\image-20220702084532455.png)

从阿里云上拉取镜像

```shell
$ docker pull registry.cn-hangzhou.aliyuncs.com/xh-images/my-registry:[镜像版本号]
```

![image-20220702085547217](img.assets\image-20220702085547217.png)



## 8.2 发布到私有库

私有库:Docker Registry是官方提供的工具，可以用于构建私有镜像仓库,官方Docker Hub地址：https://hub.docker.com/



发布到私有库步骤:

1.下载镜像Docker Registry

![image-20220702151505024](img.assets\image-20220702151505024.png)



2.运行私有库Registry，相当于本地有个私有Docker Hub

默认情况，仓库被创建在容器的`/var/lib/registry`目录下，建议自行用容器卷映射，方便于宿主机联调

`docker run -d -p 5000:5000 -v /root/myregistry/:/tmp/registry --privileged=true registry`

![image-20220702152059937](img.assets\image-20220702152059937.png)



Docker挂载主机目录访问如果出现cannot open directory .: Permission denied

解决办法：在挂载目录后多加一个`--privileged=true`参数即可

如果是CentOS7安全模块会比之前系统版本加强，不安全的会先禁止，所以目录挂载的情况被默认为不安全的行为，在SELinux里面挂载目录被禁止掉了，如果

要开启，我们一般使用--privileged=true命令，扩大容器的权限解决挂载目录没有权限的问题，也即使用该参数，container内的root拥有真正的root权限，否则，

container内的root只是外部的一个普通用户权限。



3.在ubuntu上安装ifconfig命令，并且生成新的镜像便于后续使用

```
apt-get update

apt-get install net-tools
```

![image-20220702153815370](img.assets\image-20220702153815370.png)



4.curl 验证私服库中有什么镜像

`curl -XGET http://192.168.153.140:5000/v2/_catalog`

![image-20220702154309671](img.assets\image-20220702154309671.png)



5.修改符合私服规范的Tag

`docker  tag  镜像名:Tag  Host:Port/Repository:Tag`  ------>复制＋重命名

![image-20220702154811561](img.assets\image-20220702154811561.png)



6.修改配置文件使之支持http

docker默认不允许http方式推送镜像

```
vim /etc/docker/daemon.json
```

在后面添加，其结构为json，**不要忘记逗号**

```
 "insecure-registries": ["192.168.153.140:5000"]
```

![image-20220702155352286](img.assets\image-20220702155352286.png)

修改完后如果不生效，建议重启docker

 

7.将本地镜像推送到私有库中

`docker push 192.168.153.140:5000/xhubuntu:1.0.0 `

![image-20220702160257959](img.assets\image-20220702160257959.png)





8.curl验证私服库中有什么镜像

`curl -XGET http://192.168.153.140:5000/v2/_catalog`

![image-20220702160405089](img.assets\image-20220702160405089.png)



9.pull 到本地中运行

![image-20220702160733835](img.assets\image-20220702160733835.png)



10.查看镜像的版本号

`curl http://ip:端口/v2/镜像名/tags/list`

![image-20220702161049162](img.assets\image-20220702161049162.png)



# 9.docker容器数据卷

卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性：卷的设计目的就是**数据的持久化，完全独立于容器的生存周期**，因此Docker不会在容器删除时删除其挂载的数据卷。类似Redis里面的rdb和aof文件



运行一个带有容器卷存储功能的容器实例:

`docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录 镜像名:Tag`



将运用与运行的环境打包镜像，run后形成容器实例运行 ，但是我们对数据的要求希望是持久化的 

Docker容器产生的数据，如果不备份，那么当容器实例删除后，容器内的数据自然也就没有了。为了能保存数据在docker中我们使用卷。

 

特点：

1：数据卷可在容器之间共享或重用数据

2：卷中的更改可以直接实时生效

3：数据卷中的更改不会包含在镜像的更新中

4：数据卷的生命周期一直持续到没有容器使用它为止



## 9.1 添加容器卷

命令:`docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录 镜像名:Tag`

**容器卷可以挂载多个**

![image-20220703020418542](img.assets\image-20220703020418542.png)



查看容器卷是否挂载成功:

`docker inspect 容器ID`

![image-20220703020619159](img.assets\image-20220703020619159.png)



## 9.2 读写规则映射

若直接使用`docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录 镜像名:Tag`命令，默认的权限为读写

既:`docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录:rw 镜像名:Tag`, **rw = read + write**



**设置只读规则：**

/容器目录:**ro** 镜像名        就能完成功能，此时**容器自己只能读取不能写**,**ro = read only**

`docker run -it --privileged=true`

此时如果宿主机写入内容，可以同步给容器内，容器可以读取到。

![image-20220703021855626](img.assets\image-20220703021855626.png)



## 9.3 卷的继承和共享

`docker run -it --privileged=true --volumes-from 要继承的容器名或id 镜像名`

**容器只继承父容器的容器数据卷规则和其中的数据，两个容器之间没有关系，即使父容器停止，不会影响到改容器**

![image-20220703023228256](img.assets\image-20220703023228256.png)



