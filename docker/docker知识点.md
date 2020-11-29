# 一、Docker介绍

# 1.1、容器

容器是对应用程序及其依赖关系的封装，与虚拟机一样拥有一个被隔离的操作系统实例，用来运行应用程序。

容器与虚拟机相比，具有以下优点：

* 容器能与主机的操作系统共享资源，因而它的效率高出一个数量级；
* 容器具有可移植性；
* 容器是轻量的；

另外，虚拟机和容器的根本目标不尽相同。虚拟机的目的是要完整地模拟另一个环境，而容器的目的则是使应用程序能够移植，并把所有依赖关系包含进去。

# 1.2、Docker

Docker利用现有的Linux容器技术，以不同方式将其封装及扩展——主要是通过提供可移植的镜像，以及一个用户友好的接口——来创建一套完整的容器创建及发布方案。Docker平台拥有两个不同的部分：负责创建与运行容器的Docker引擎，以及用来发布容器的云服务Docker Hub。Docker引擎提供了一个快速且便携的接口用来运行容器。

# 1.3、基本命令

docker run：负责启动容器；

```
docker run -i -t debian /bin/bash
```

当中的-i和-t参数表示我们想要一个附有tty的交互会话，/bin/bash参数表示你想获得一个bash shell。

docker inspect：检查容器；

```java
docker inspect modest_kilby |grep IPAddress
```

检查名称为modest_kilby的容器，过滤出它的IP地址来。

docker diff：找出容器有哪些文件改动过；

```
docker diff modest_kilby
```

# 1.4、通过Dockerfile创建镜像

```
FROM debian:wheezy
// FROM指令指定初始镜像（这里使用Debian，并且指定使用wheezy版本）
RUN apt-get update && apt-get install -y cowsay fortune
```

所有Dockerfile一定要有FROM指令作为第一个非注释指令。RUN指令指定的shell命令，是将要在镜像里执行的。

创建及运行：

```
docker build -t test/cowsay-dockerfile .
docker run test/cowsay-dockerfile /usr/games/cowsay "Moo"
```

五种容器状态：已创建（created）、重启中（restarting）、运行中（running）、已暂停（paused）和已退出（exited）。

通过利用Dockerfile的ENTRYPOINT指令，可以让用户更易于使用这个镜像。ENTRYPOINT指令让我们指定一个可执行文件，同时还能处理传给docker run的参数。

在Dockerfile的最后加上一行：

```
ENTRYPOINT ["/usr/games/cowsay"]
```

加完之后，上面的创建及运行命令可修改为：

```
docker build -t test/cowsay-dockerfile .
docker run test/cowsay-dockerfile "Moo"
```

ENTRYPOINT还可以指定为一个我们自己的脚本。

# 2.1、基本概念

## 2.1.1、底层技术

Docker守护进程通过一个“执行驱动程序”来创建容器。

* cgroups，负责管理容器使用的资源（例如CPU和内存的使用）。它还负责冻结和解冻容器这两个docker pause命令所需的功能；
* Namespaces（命名空间），负责容器之间的隔离；它确保系统的其他部分与容器的文件系统、主机名、用户名、网络和进程都是分开的。

另一个主要的Docker底层技术就是联合文件系统。

服务发现

当Docker容器启动时，它需要通过某种方法来找出与之通信的服务，而这些服务一般也是运行在容器内的。

服务编排及集群管理

对每个新的容器，都需要安排把它放置在哪台主机上，并对它实施监控和保持更新。系统需要对故障的出现或负载的改变作出反应，实际情况下可能是搬迁、启动或停止容器。

docker build命令需要Dockerfile和构建环境的上下文。上下文是一组本地文件和目录，它可以被Dockerfile的ADD或COPY指令所引用，通常以目录路径的形式指定。该目录下的所有文件和目录就形成了构建环境的上下文，并在生成镜像的过程中传递给Docker守护进程。

当Dockerfile指令成功执行后，中间使用过的那个容器会被删掉，除非提供了--rm=false参数。由于每个指令的最终结果都只是个静态的镜像而已，本质上除了一个文件系统和一些元数据以外就没有其他东西，因此即使指令中有任何正在运行的进程，最后都会被停掉。

**镜像实际可以理解为一些数据。**

Docker为了加快镜像构建的速度，也会将每一个镜像层缓存下来。Docker的缓存特性能大大提高工作效率。

如果你需要使缓存失效，可以在执行docker build的时候加上--no-cache参数。

当你创建自己的镜像时，你需要决定从哪个基础镜像开始。

连接的初始化是通过执行docker run命令时传入--link CONTAINER:ALIAS参数，其中CONTAINER是目标容器的名称，而ALIAS是主容器用来称呼目标容器的一个本地名称。



**docker这部分不知道怎么写，先写这么多，后面根据面试或者查看博客再进行扩展，应该就是记住一些指令比较关键，然后就是对容器的一个理解，理解它和虚拟机的区别，理解为何要使用容器，有什么好处。**