---
title: Docker的学习（二）----构建镜像
date: 2017-08-13 10:47:45
tags:
    - Docker
---

在本篇中，我将记录对Docker镜像深入的学习，已经对Dockerfile的编写和理解。

###### Docker镜像
>Dcoker镜像是由文件系统叠加而成的。最底端是一个引导文件系统，即bootfs。

###### 构建镜像
在学习过程中，发现构建方法有两种：

>使用`docker commit`命令    
使用`docker build`命令和Dockerfile文件

* 使用Docker的commit命令构建镜像

    类似Git中的commit，要求先创建一个容器，并在容器中进行修改，然后将修改提交创建为一个新的镜像。

    Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

* 使用Dockfile构建镜像

    Dockerfile使用基本的基于DSL语法的指令来构建一个Docker镜像，之后使用`docker build`命令基于Dockerfile中的指令来构建一个新的镜像。    Dockerfile是Docker构建镜像的基础，也是Docker区别于其他容器的重要特征，正是有了Dockerfile，Docker的自动化和可移植性才成为可能。    
    Dockerfile分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

###### Dockerfile 关键字

* **FROM**

该命令定义了使用哪个基础镜像启动构建流程。基础镜像可以为任意镜像。FROM指令指定的基础image可以是官方远程仓库中的，也可以位于本地仓库。如果基础镜像没有被发现，Docker将试图从Docker image index来查找该镜像。FROM命令必须是Dockerfile的首个命令。

Usage:

`FROM <image>`

`FROM <image>:<tag>`

`FROM <image>@<digest>`

* **MAINTAINER**

维护者信息

Usage:

`MAINTAINER <name>`

* **RUN**

非交互式运行shell命令，RUN可以运行任何被基础image支持的命令，可以有多条。

RUN指令会在一个新的容器中执行任何命令，然后把执行后的改变提交到当前镜像，提交后的镜像会被用于Dockerfile中定义的下一步操作，RUN中定义的命令会按顺序执行并提交。当命令较长时可以使用`\`来换行。

Usage:

`RUN <command> (the command is run in a shell)`

`RUN ["executable", "param1", "param2" ... ]  (exec form)`

* **CMD**

Container启动时执行的命令，但是一个Dockerfile中只能有一条CMD命令，多条则只执行最后一条CMD.

CMD主要用于Container时启动指定的服务，当`Docker run command`的命令匹配到CMD command时，会替换CMD执行的命令。

Usage:

`CMD ["executable","param1","param2"] (exec form, this is the preferred form)`

`CMD ["param1","param2"] (as default parameters to ENTRYPOINT)`

`CMD command param1 param2 (shell form)`

* **ENTRYPOINT**

Container启动时执行的命令，但是一个Dockerfile中只能有一条ENTRYPOINT命令，如果多条，则只执行最后一条。

ENTRYPOINT没有CMD的可替换特性。

该指令的使用分为两种情况，一种是独自使用，另一种和CMD指令配合使用。

当独自使用时，如果你还使用了CMD命令且CMD是一个完整的可执行的命令，那么CMD指令和ENTRYPOINT会互相覆盖只有最后一个CMD或者ENTRYPOINT有效。

另一种用法和CMD指令配合使用来指定ENTRYPOINT的默认参数，这时CMD指令不是一个完整的可执行命令，仅仅是参数部分；ENTRYPOINT指令只能使用JSON方式指定执行命令，而不能指定参数。

Usage:

`ENTRYPOINT ["executable", "param1", "param2"] (exec form, preferred)`

`ENTRYPOINT command param1 param2 (shell form)`

* **ADD**

ADD只有在build镜像的时候运行一次，后面运行Container的时候不会再重新加载了。

将外部文件拷贝到镜像里,src可以为url。

将文件<src>拷贝到container的文件系统对应的路径<dest>。

所有拷贝到Container中的文件和文件夹权限为0755,uid和gid为0。

有下载URL和解压的功能。

如果文件是可识别的压缩格式，则docker会帮忙解压缩。

如果要ADD本地文件/远程文件，则本地文件/远程文件必须在 docker build <PATH>，指定的<PATH>目录下。

需要自动下载URL并拷贝到Container时使用

Usage:

`ADD <src>... <dest>`

`ADD ["<src>",... "<dest>"] (this form is required for paths containing whitespace)`

><src> 是相对被构建的源目录的相对路径，可以是文件或目录的路径，也可以是一个远程的文件url;
<dest> 是container中的绝对路径

* **COPY**

没有下载URL和解压的功能。

不需要自动下载URL并拷贝到Container时使用。

复制本地主机的 <src> （为Dockerfile所在目录的相对路径）到容器中的 <dest>。

当使用本地目录为源目录时，推荐使用COPY。

Usage:

`COPY <src>... <dest>`

`COPY ["<src>",... "<dest>"] (this form is required for paths containing whitespace)`


* **ENV**

在Image中设置一个环境变量。

设置了后，后续的RUN命令都可以使用，container启动后，可以通过docker inspect查看这个环境变量，也可以通过在docker run --env key=value时设置或修改环境变量。

Usage:

`ENV <key> <value>`

`ENV <key>=<value> ...`

* **WORKDIR**

切换工作目录用，可以多次切换(相当于cd命令)，对RUN,CMD,ENTRYPOINT生效

Usage:

`WORKDIR /path/to/workdir`

* **USER**

设置启动容器的用户，默认是root用户。

USER指令用于设置用户或uid来运行生成的镜像和执行RUN、CMD、ENTRYPOINT。

Usage:

`USER daemon`

* **LABEL**

用键值对的方式来指定image的元数据

Usage:

`LABEL <key>=<value> <key>=<value> <key>=<value> ...`

* **EXPOSE**

container内部服务开启的端口。

指定在docker允许时指定的端口进行转发。

主机上要用还得在启动Container时，做Host-Container的端口映射

Usage:

`EXPOSE <port> [<port>...]`

* **VOLUME**

Usage:

使容器中的一个目录具有持久化存储数据的功能，该目录可以被容器本身使用，也可以共享给其他容器使用。

我们知道容器使用的是AUFS，这种文件系统不能持久化数据，当容器关闭后，所有的更改都会丢失。

当容器中的应用有持久化数据的需求时可以在Dockerfile中使用该指令。

创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

`VOLUME ["/data"]`

* **ARG**

ARG是Docker1.9 版本才新加入的指令。

ARG 定义的变量只在建立Image时有效，建立完成后变量就失效消失。

Usage:

`ARG <name>[=<default value>]`

* **ONBUILD**

ONBUILD 的作用就是让指令延迟執行，延迟到下一个使用 FROM 的 Dockerfile 在建立Image时执行，只限延迟一次。

ONBUILD 的使用情景是在建立镜像时取得最新的源码 (搭配 RUN) 与限定系统框架。

Usage:

`ONBUILD [INSTRUCTION]`


###### 第一个Dockerfile

```
# Base ubuntu 14.04 image
# VERSION 0.0.1
# Authon: Ken

# 基础镜像信息
FROM ubuntu:14.04

# 维护者信息
MAINTAINER Ken ken.han.coder@aliyun.com

# 镜像操作指令
RUN mv /etc/apt/sources.list /etc/apt/sources.list.1
# 更新源文件
COPY sources.list /etc/apt/sources.list

RUN apt-get -y update
RUN apt-get -y upgrade
RUN apt-get -y install vim curl wget
```

###### 使用docker build
Usage:  docker build [OPTIONS] PATH | URL | -

`docker build -t="ken_repo/ubuntu:14.04_64_base_image" .`

```
REPOSITORY          TAG                   IMAGE ID            CREATED             SIZE
ken_repo/ubuntu     14.04_64_base_image   583b4e36d909        15 seconds ago      294.1 MB
ubuntu              16.10                 175e129b1641        2 weeks ago         100.1 MB
ubuntu              latest                42118e3df429        2 weeks ago         124.8 MB
ubuntu              14.04                 0ccb13bf1954        2 weeks ago         188 MB
```
