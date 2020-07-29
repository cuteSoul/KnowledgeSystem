# Docker基础

官网地址：https://www.docker.com/

文档地址：https://docs.docker.com/

仓库地址：https://hub.docker.com/

## Docker概述

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的 Linux或Windows 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

![img](https://images2018.cnblogs.com/blog/1011251/201806/1011251-20180609113423222-152077047.png)

> Docker是开发人员和系统管理员使用容器开发、部署和运行应用程序的平台。使用Linux容器来部署应用程序称为集装箱化。使用docker轻松部署应用程序。

集装箱化的优点：

- 灵活：即使是复杂的应用程序也可封装。
- 轻量级：容器利用并共享主机内核。
- 便携式：您可以在本地构建，部署到云上并在任何地方运行。
- 可扩展性：您可以增加和自动分发容器副本。
- 可堆叠：您可以垂直堆叠服务并及时并及时堆叠服务。

### Docker为什么出现？

一款产品：开发～上线两套环境！应用环境，应用配置！

开发 ---- 运维。问题：我在我的电脑上可以运行！版本更新，导致服务不可用！对于运维来说，考验就十分大？

环境配置是十分的麻烦，每一个机歇绪趣要部署环境（集群 Redis 、 ES 、 Had oo p  … ）！费时费力

发布一个项目（ jar + ( Redis MySQL jdk ES ) ) ，项目能不能都带上环境安装打包！

之前在服务器配置一个应用的环境 Redis MysQL ) dk ES Hodoop ，配置超麻烦了，不能够跨平台。

Windows ，最后发布到 Linux！

传统：开发jar ，运维来做！

现在：开发打包部署上线，一套流程做完！



 Java -- apk  -- 发布（应用商店）---- 张三使用apk -- 装即可用！ 

Java -- jar （环境） ----  打包项目带上环境（镜像） ----（ Docker 仓库：商店） -- 下载我们发布的镜像  --  直接运行即可 ！

Docker 给以上的问题，提出了解决方案！

![image-20200729104257698](https://cdn.jsdelivr.net/gh/cuteSoul/imgbed/img/image-20200729104257698.png)

Docker运用隔离机制，可以将服务器利用到极致！

### Docker历史

**lxc --> libcontainer --> runC**

> docker在起步阶段使用lxc，但发现其并不是那么好用，于是docker公司就自己把cgroups和lxc做了一个联合封装。lxc向上提供的接口做了二次封装，使其变成更易用的容器管理界面和容器管理逻辑。从而使得整个容器的使用变得更加简便了。

容器的核心组件:

- **NameSpaces** :名称空间

> **仅能做到隔离，而不能做到资源分配的问题。PID,user,hostname,network都属于内核的，所以需要隔离开来。**
>
> **rootfs: 根文件系统(mount)**
>
> **PID**
>
> **user**
>
> **hostname** /proc/system/hostname
>
> **network**
>
> **ipc : 进程间通信(Inter-Process Communication)**

- **Control Groups**: 控制组

> **他能够实现以进程为单位，来尝试着把我们的CPU的时间分配，用量分配，内存空间分配，IO用量分配以指定比例或指定数量的方式分派给多个进程。**

- **Chroot** : 为了安全的目的等如Selinux

### Docker用处

![image-20200729105515086](https://cdn.jsdelivr.net/gh/cuteSoul/imgbed/img/image-20200729105515086.png)

虚拟机技术的缺点：

* 资源占用多
* 冗余步骤多
* 启动慢

> 容器化技术

==容器化技术不是模拟的一个完整的操作系统==

![image-20200729105738284](https://cdn.jsdelivr.net/gh/cuteSoul/imgbed/img/image-20200729105738284.png)

不同：

* 传统虚拟机，虚拟出一套硬件，运行一个完整的操作系统，然后再这个系统上安装和运行软件
* 容器内的应用直接运行在宿主机的内核中，容器是没有自己内核的，也没有虚拟出硬件，所以很轻便
* 每个容器间是互相隔离，每个容器内都有一个属于自己的文件系统，互不影响。

> DevOps（开发、运维）

**应用更快速的交付和部署**

传统：一堆帮助文档，安装程序

Docker：打包镜像发布测试，一键运行

**更便捷的升级和扩缩容**

使用Docker，部署容器相当于搭积木一样简便

项目打包成一个镜像，可扩展性强。

**更简单的系统运维**

在容器化之后，开发、测试、部署都是高度一致

**更高效的计算资源利用**

Docker是内核级别的虚拟化，可以在一个物理机上运行很多个容器！服务器性能可以被利用到极致。

## Docker安装

### Docker的基本组成

![image-20200729111006720](https://images2018.cnblogs.com/blog/1202606/201802/1202606-20180221171842775-1155015821.png)

>  **镜像（images）：**

docker镜像相当于一个模板，可以通过这个模板来创建容器服务且可以创建多个容器（最终服务运行或者项目的运行就是在容器中）。

Docker镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数(如匿名卷、环境变量、用户等)。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

> 容器（container）

镜像(image)和容器(container)的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间。

> 仓库（repository）

存放镜像的地方！

仓库分为共有和私有仓库！

### 安装Docker

> 环境准备

| Platform                                                   | x86_64 / amd64                                               | ARM                                                          | ARM64 / AARCH64                                              |
| :--------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| [CentOS](https://docs.docker.com/engine/install/centos/)   | [![yes](https://docs.docker.com/images/green-check.svg)](https://docs.docker.com/engine/install/centos/) |                                                              | [![yes](https://docs.docker.com/images/green-check.svg)](https://docs.docker.com/engine/install/centos/) |
| [Debian](https://docs.docker.com/engine/install/debian/)   | [![yes](https://docs.docker.com/images/green-check.svg)](https://docs.docker.com/engine/install/debian/) | [![yes](https://docs.docker.com/images/green-check.svg)](https://docs.docker.com/engine/install/debian/) | [![yes](https://docs.docker.com/images/green-check.svg)](https://docs.docker.com/engine/install/debian/) |
| [Fedora](https://docs.docker.com/engine/install/fedora/)   | [![yes](https://docs.docker.com/images/green-check.svg)](https://docs.docker.com/engine/install/fedora/) |                                                              | [![yes](https://docs.docker.com/images/green-check.svg)](https://docs.docker.com/engine/install/fedora/) |
| [Raspbian](https://docs.docker.com/engine/install/debian/) |                                                              | [![yes](https://docs.docker.com/images/green-check.svg)](https://docs.docker.com/engine/install/debian/) | [![yes](https://docs.docker.com/images/green-check.svg)](https://docs.docker.com/engine/install/debian/) |
| [Ubuntu](https://docs.docker.com/engine/install/ubuntu/)   | [![yes](https://docs.docker.com/images/green-check.svg)](https://docs.docker.com/engine/install/ubuntu/) | [![yes](https://docs.docker.com/images/green-check.svg)](https://docs.docker.com/engine/install/ubuntu/) | [![yes](https://docs.docker.com/images/green-check.svg)](https://docs.docker.com/engine/install/ubuntu/) |

> 安装 （以centos为例）

参考：https://docs.docker.com/engine/install/centos/

```shell
# 1、卸载旧的版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
# 2、安装所需的包
yum install -y yum-utils

# 3、设置镜像仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo   # 国外的很慢
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo   # 阿里云镜像地址

# 更新yum软件包索引
yum makecache fast

# 4、安装Docker相关
yum install docker-ce docker-ce-cli containerd.io

# 5、启动docker
systemctl start docker

# 6、测试
docker run hello-world

# 7、查看镜像
docker images
```

删除docker：

```shell
# 1、卸载依赖
yum remove docker-ce docker-ce-cli containerd.io

# 2、删除资源
rm -rf /var/lib/docker
```

###  回顾Hello World流程

![image-20200729133800540](https://cdn.jsdelivr.net/gh/cuteSoul/imgbed/img/image-20200729133800540.png)

### 底层原理

**Docker**是一个**Client-Server**结构的系统，**Docker**守护进程运行在主机上，然后通过S**ocket**连接从客户端访问， 守护进程从客户端接受命令并管理运行在主机上的容器。容器，是一个运行时环境，就是我们前面说到的集装箱。

![image-20200729134115995](https://cdn.jsdelivr.net/gh/cuteSoul/imgbed/img/image-20200729134115995.png)

(1)**docker**有着比虚拟机更少的抽象层。由宁**docker**不需要**Hypervisor**实现硬件资源虚拟化,运行在**docker**容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上**docker**将会在效率.上有明显优势。

(2)**docker**利用的是宿主机的内核，而不需要**Guest OS**。因此，当新建一个容器时,**docker**不需要和虚拟机那样重新加载一个操作系统内核。仍而避免引寻、加载操作系统内核返个比较费时费资源的过程,当新建一-个虚拟机时,虚拟机软件需要加载**GuestOS**,返个新建过程是分钟级别的。而**docker**由于直接利用宿主机的操作系统，则省略了返个过程,因此新建一个**docker**容器只需要几秒钟。

![img](https://images2018.cnblogs.com/blog/1011251/201806/1011251-20180610171525142-1923858765.png)

## Docker常用命令

帮助文档地址：https://docs.docker.com/engine/reference/commandline

### 基本命令

`docker version`		  查看docker版本
`docker info`				查看docker详细信息
`docker --help`			帮助命令

### 镜像命令

`docker images`			查看镜像

```shell
PEPOSITORY：	镜像的仓库源
TAG：		镜像的标签
IMAGE ID：	镜像ID
CREATED：	镜像创建时间
SIZE：		镜像大小
```

`docker images -a`		列出本地所有的镜像
`docker images -p`		只显示镜像ID
`docker images --digests`		显示镜像的摘要信息
`docker images --no-trunc`		显示完整的镜像信息

`docker search`			从Docker Hub上查找镜像

`docker search mysql --filter=STARS=3000`			搜索过程加过滤

`docker pull 镜像名[:tag]` 			下载镜像，可加版本号

`docker commit -m "提交的描述信息" -a "作者" 容器ID 要创建的目标镜像名称:[标签名]`提交容器使之成为一个新的镜像

如：`docker commit -m "新的tomcat" -a "lizq" f9e29e8455a5 mytomcat:1.2`
`docker rmi hello-world`		从Docker中删除hello-world镜像
`docker rmi -f hello-world`		从Docker中强制删除hello-world镜像
`docker rmi -f hello-world nginx`		从Docker中强制删除hello-world镜像和nginx镜像
`docker rmi -f $(docker images -ap)`通过`docker images -ap`		查询到的镜像ID来删除所有镜像

### 容器命令

说明：有了镜像才能创建容器。

`docker run [OPTIONS] IMAGE`		根据镜像新建并启动容器。IMAGE是镜像ID或镜像名称

```
OPTIONS说明：
 --name=“容器新名字”：为容器指定一个名称
 -d：后台运行容器，并返回容器ID，也即启动守护式容器
 -i：以交互模式运行容器，通常与-t同时使用
 -t：为容器重新分配一个伪输入终端，通常与-i同时使用
 -P：随机端口映射
 -p：指定端口映射，有以下四种格式：
  ip:hostPort:containerPort
  ip::containerPort
  hostPort:containerPort
  containerPort
```

`docker ps`				  列出当前所有正在运行的容器
`docker ps -a`			列出所有的容器
`docker ps -l`			列出最近创建的容器
`docker ps -n 3`		列出最近创建的3个容器
`docker ps -q`			只显示容器ID
`docker ps --no-trunc`			显示当前所有正在运行的容器完整信息
`exit`							退出并停止容器
`Ctrl+p+q`					只退出容器，不停止容器
`docker start 容器ID或容器名称`				  启动容器
`docker restart 容器ID或容器名称`			  重新启动容器
`docker stop容器ID或容器名称`					  停止容器
`docker kill 容器ID或容器名称`					强制停止容器
`docker rm 容器ID或容器名称`						删除容器
`docker rm -f 容器ID或容器名称`				  强制删除容器
`docker rm -f $(docker ps -a -q)`   	  删除多个容器
`docker logs -f -t --since --tail 容器ID或容器名称`			查看容器日志

```
如：docker logs -f -t --since=”2018-09-10” --tail=10 f9e29e8455a5
 -f : 查看实时日志
 -t : 查看日志产生的日期
 --since : 此参数指定了输出日志开始日期，即只输出指定日期之后的日志
 --tail=10 : 查看最后的10条日志
```

`docker top 容器ID或容器名称`				    查看容器内运行的进程
`docker inspect 容器ID或容器名称`			查看容器内部细节
`docker attach 容器ID`								进到容器内
`docker exec 容器ID`									进到容器内
`docker cp 容器ID:容器内的文件路径 宿主机路径`					从容器内拷贝文件到宿主机
如：`docker cp f9e29e8455a5:/tmp/yum.log  /root`

### Docker命令小结

![image-20200729141142006](https://cdn.jsdelivr.net/gh/cuteSoul/imgbed/img/image-20200729141142006.png)







