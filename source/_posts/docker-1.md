---
title: Docker的学习（一）----入门
date: 2017-08-13 10:47:12
tags:
    - Docker
---

之前有大致了解学习过Docker的基础操作，后来由于生活、工作搁置了对它的学习。现在要重拾起来，从头再学习，并记录在Blog中，以便复习。

### 简介
每学一门新技术，都必须了解它的起源、作用。简介内容摘自网上，保存用于祥读。

docker的英文本意是码头工人，也就是搬运工，这种搬运工搬运的是集装箱（Container），集装箱里面装的可不是商品货物，而是任意类型的App，Docker把App（叫Payload）装在Container内，通过Linux Container技术的包装将App变成一种标准化的、可移植的、自管理的组件，这种组件可以在你的latop上开发、调试、运行，最终非常方便和一致地运行在production环境下。

Docker的核心底层技术是LXC（Linux Container），Docker在其上面加了薄薄的一层，添加了许多有用的功能。

Docker提供了一种可移植的配置标准化机制，允许你一致性地在不同的机器上运行同一个Container；而LXC本身可能因为不同机器的不同配置而无法方便地移植运行；

Docker以App为中心，为应用的部署做了很多优化，而LXC的帮助脚本主要是聚焦于如何机器启动地更快和耗更少的内存；

Docker为App提供了一种自动化构建机制（Dockerfile），包括打包，基础设施依赖管理和安装等等；

Docker提供了一种类似git的Container版本化的机制，允许你对你创建过的容器进行版本管理，依靠这种机制，你还可以下载别人创建的Container，甚至像git那样进行合并；

Docker Container是可重用的，依赖于版本化机制，你很容易重用别人的Container（叫Image），作为基础版本进行扩展；

Docker Container是可共享的，有点类似github一样，Docker有自己的INDEX，你可以创建自己的Docker用户并上传和下载Docker Image；

Docker提供了很多的工具链，形成了一个生态系统；这些工具的目标是自动化、个性化和集成化，包括对PAAS平台的支持等；

那么Docker有什么用呢？对于运维来说，Docker提供了一种可移植的标准化部署过程，使得规模化、自动化、异构化的部署成为可能甚至是轻松简单的事情；而对于开发者来说，Docker提供了一种开发环境的管理方法，包括映像、构建、共享等功能，而后者是本文的主题。

Docker能处理的事情包括：

* 隔离应用依赖
* 创建应用镜像并进行复制
* 创建容易分发的即启即用的应用
* 允许实例简单、快速地扩展
* 测试应用并随后销毁它们

Docker三大核心概念

* 镜像 Image ：类似于虚拟机的快照，但更轻量。
* 容器 Container ：镜像的一个运行实例，可以独立运行一个或一组应用。
* 仓库 Repository ：集中存放镜像的地方


### Mac 安装
> 系统： Mac X 10.11.6

我参考了官方文档，[地址](https://docs.docker.com/docker-for-mac/)

Mac安装Docker很简单，只要下载安装包Docker.dmg，就行了。


### Linux 安装
> 系统：CentOS 6.8 64位

由于VPS系统内核为2.6，不满足运行Docker的要求，需要先进行内核升级。

内核升级参考[地址](https://segmentfault.com/a/1190000000733628)

然后就sad了，我目前用的是搬瓦工的VPS，它是OpenVZ，内核没法升级，这怎么搞，这没法搞啊。

不过之前有在Ubuntu上安装过，具体还是参考[官方文档](https://docs.docker.com/engine/installation/linux/ubuntulinux/)


### 使用过程
为了解决国内使用官方Docker Hub时遇到的稳定性及速度问题，我使用了DaoCloud提供的加速器服务。详细配置过程见[官方文档](https://www.daocloud.io/mirror.html#accelerator-doc)

官方Docker Hub 其实就是个Registry，但是它是公有的。它提供存储镜像数据，并且提供拉取和上传镜像的功能。

* 镜像

    * 获取镜像

        Usage:  docker pull [OPTIONS] NAME[:TAG|@DIGEST]

        ```
        docker pull ubuntu:16.10
        ```

        命令分析：

        ubuntu为镜像名称，'16.10'为Tag
    
    * 删除镜像

        Usage:  docker rm [OPTIONS] CONTAINER [CONTAINER...]

        ```
        docker rm 42118e3df429
        ```

    * 获取镜像列表

        ```
        docker images
        ```

```
    → docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ubuntu              16.10               175e129b1641        2 weeks ago         100.1 MB
    ubuntu              latest              42118e3df429        2 weeks ago         124.8 MB
```

上面中ubuntu并不是镜像名称，而是代表了一个名为ubuntu的Repository，同时在这个Repository下面有一系列打了Tag的Image，IMAGE ID 是一个GUID，为了方便也可以通过Repository:tag来引用。IMAGE ID相同说明Tag指向了同一个镜像文件。


* 容器

    * 运行第一个容器

        使用 ```docker run ```命令

        Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

        ```
        docker run --name first_container -it ubuntu /bin/bash
        ```

        命令分析：

        >--name string      Assign a name to the container
        -i, --interactive    Keep STDIN open even if not attached    
        -t, --tty        Allocate a pseudo-TTY

        --name的作用是为容器分配一个名称，可以用容器的名称来替代容器ID。容器名称有助于分辨容器，当构建容器和应用程序之间的逻辑连接时，容器的名称也有助于从逻辑上理解连接关系。

        -i的作用是保证容器中得STDIN（标准输入）是开启的，甚至并没有附着在容器上，从而保证了持久的标准输入。

        -t的作用是让Docker为被创建的容器分配一个伪tty，这样新建的容器才能提供一个交互式shell。

        /bin/bash 是告诉Docker要在容器中运行/bin/bash命令，从而启动一个Bash Shell。

        在网上教材中会有使用 ```ip a``` 或者 ```ifconfig``` 来检查容器的网络接口，但是在我的容器中 ``` bash: ifconfig: command not found ``` 。其实是由于设计哲学就是不推荐里面设IP，所以系统中就没有安装。

        输入命令 ```exit``` 就可以退出容器返回到宿主机上。此时容器已经停止，因为只有在指定的/bin/bash命令处于运行状态的时候，我们容器也才会相应地处于运行状态。一旦退出容器，/bin/bash命令也就结束了，这时容器也随之停止了运行。

    * 获取容器列表

        Usage:  docker ps [OPTIONS]

        * 获取当前正在运行的容器列表

            ```
            docker ps
            ```

        * 获取当前系统所有的容器列表

            ```
            docker ps -a
        ```

    * 容器的启动与停止

        >start     Start one or more stopped containers
        stop      Stop one or more running containers

        ``` docker start CONTAINER_NAME/CONTAINER_ID ``` 根据容器名称/容器ID来启动容器

        ``` docker stop CONTAINER_NAME/CONTAINER_ID ``` 根据容器名称/容器ID来停止容器

    * attach

         >attach   Attach to a running container

        Usage:  docker attach [OPTIONS] CONTAINER

         ```docker attach 0406e38d7fea``` 可以附着到一个容器ID为0406e38d7fea的并且正在运行的容器。

    * exec

        > exec      Run a command in a running container

        Usage:  docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

        通过docker exec命令在容器内部额外启动新进程。可以在容器内运行的进程有两种类型：后台任务和交互式任务。后台任务在容器内运行且没有交互需求，而交互式任务则保持在前台运行。对于需要在容器内部打开shell的任务，交互式任务是很实用的。

        * 后台任务

            >-d, --detach   Detached mode: run command in the background

            ```docker exec -d 0406e38d7fea touch ~/test```

            命令分析：

            -d的作用是开启后台模式，将命令行命令运行在后台。该命令的作用是在'~/'目录下创建了个test文件

        * 交互式任务

            >-i, --interactive    Keep STDIN open even if not attached
            -t, --tty            Allocate a pseudo-TTY

            docker exec -it 0406e38d7fea /bin/bash

    * 容器的删除

        Usage:  docker rm [OPTIONS] CONTAINER [CONTAINER...]

        ```
        docker rm 0406e38d7fea
        ```

        >运行中的Docker容器是无法删除的
