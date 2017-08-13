---
title: Docker的学习（三）----使用Dockerfile部署一个Flask项目
date: 2017-08-13 10:45:30
tags: 
    - Docker
---


建立了一个基础Ubuntu镜像、Nginx镜像及Python镜像。

使用了docker-compose提供堆栈完成多容器的组装，完成部署一个Flask项目。

项目的Github地址：https://github.com/kenhancoder/docker_repo

##### 基础Ubuntu镜像
创建了基础镜像repo/ubuntu:16.04_64_Base，镜像基于phusion/baseimage:0.9.19 [Github地址](!https://github.com/phusion/baseimage-docker)

Dockerfile如下
```
# 基础镜像信息
FROM phusion/baseimage:0.9.19

# Use baseimage-docker's init system.
CMD ["/sbin/my_init"]

# ...put your own build instructions here...

# 时区
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# 清华大学源
RUN mv /etc/apt/sources.list /etc/apt/sources.list.1
COPY sources.list /etc/apt/sources.list

RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y \
                vim \
                curl \
                wget \
                build-essential \
                python-software-properties \
                python-dev \
                python-pip \
                supervisor

RUN mkdir /etc/service/supervisor
ADD supervisor.sh /etc/service/supervisor/run

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

运行`docker build -t='repo/ubuntu:16.04_64_Base' .`

##### Nginx镜像

Dockerfile如下
```
# 基础镜像信息
FROM repo/ubuntu:16.04_64_Base

# 维护者信息
MAINTAINER Ken ken.han.coder@aliyun.com

RUN add-apt-repository -y ppa:nginx/stable
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y nginx

COPY nginx.conf /etc/nginx/conf

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

运行`docker build -t='repo/nginx:1.10.1_Base' .`

##### Python镜像

Dockerfile如下
```
# 基础镜像信息
FROM repo/ubuntu:16.04_64_Base

# 维护者信息
MAINTAINER Ken ken.han.coder@aliyun.com

# 镜像操作指令
COPY ./conf/pip.conf /root/.pip/pip.conf 
ADD requirements.txt requirements.txt

RUN pip install --upgrade pip
RUN pip install -r requirements.txt

RUN mkdir -p /var/www
COPY app.py /var/www/app.py

WORKDIR /var/www

EXPOSE 8765

```

Flask代码如下
```
# -*- coding: utf-8 -*-

from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=8765)
```

运行`docker build -t="repo/flask_demo:0.1" .`

> host一定不要用默认的"127.0.0.1"，不然容器启动，即使映射了端口，在浏览器中也仍然是无法访问服务的。
> 将host设置为"0.0.0.0"，这样Flask容器可以接受到宿主的请求。

nginx配置文件如下

```
server {
    listen  8765;

    location / {
        proxy_set_header Access-Control-Allow-Origin *;
        proxy_pass_header Server;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_pass http://web:8765;
    }
}
```

> proxy_pass中的'web'为docker-compose.yml中的links的web

#####Docker Compose
使用Docker Compose，可以用一个YAML文件定义一组要启动的容器，以及容器运行时的属性。Docker Compose称这些容器为“服务”，像这样定义：`容器通过某些方法并指定一些运行时的属性来和其他容器产生交互`。

可以在一个文件中定义一个多容器的应用，然后使用一条命令来启动你的应用，然后所有相关的操作都会被自动完成

docker-compose.yml如下

```
server:
  restart: always
  image: repo/nginx:1.10.1_Base
  volumes:
    - ./conf/nginx_flask_demo.conf:/etc/nginx/conf.d/flask_demo.conf
  links:
    - web
  ports:
    - "8765:8765"

web:
  restart: always
  image: repo/flask_demo:0.1
  working_dir: /var/www
  expose:
    - "8765"
  command: python /var/www/app.py
```
> 可以将command中的启动命令换成gunicorn来启动。

##### 启动
在docker-compose.yml的同级目录下运行`docker-compose up -d`

##### 总结

###### Dockerfile
* RUN和CMD都是执行命令，他们的差异在于RUN中定义的命令会在执行docker build命令创建镜像时执行，而CMD中定义的命令会在执行docker run命令运行镜像时执行，另外使用第一种语法也就是调用exec执行时，命令必须为绝对路径。

* 使用`Docker build`命令时，可以使用`-f`参数来选择指定的Dockerfile。如`docker build -t="repo/server_flask:local_0.1" -f="Dockerfile_local" .`

###### Docker Compose
* 我在YAML文件大多使用已经创建好的镜像，即使用`image`参数。Docker Compose也提供参数`build`，参数值为Dockerfile的目录路径，所以只能将Dockerfile文件命名为"Dockerfile"，缺少灵活性。

* 使用`-f`参数可以选择指定docker-compose.yml，例如`docker-compose -f docker-compose-local.yml up -d`，这将根据docker-compose-local.yml来启动容器。
