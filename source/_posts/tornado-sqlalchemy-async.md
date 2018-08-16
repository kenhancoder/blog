---
title: 对Tornado异步操作Sqlalchemy方法的选定
date: 2017-08-13 11:05:12
tags:
    - Tornado
    - Sqlalchemy
    - Celery
    - RabbitMQ
---

### 使用原因

在一个实时通讯的项目中，由于需要使用Websocket这一协议，便在Python框架中选定了Tornado，也同时使用了Sqlalchemy这一ORM框架。
大家都知道Tornado有异步非阻塞特性，但Sqlalchemy是同步操作，这会大大影响性能，会影响的用户体验。
为了能解决这一问题，我便在网上搜寻资料，发现有使用Celery的，有使用run_on_executor装饰器的，甚至自己封装异步Sqlalchemy的等等方法。
由于缺少实践，我觉定对Celery、run_on_executor进行尝试
### Celery

以下是官方文档的介绍：
>Celery 是一个简单、灵活且可靠的，处理大量消息的分布式系统，并且提供维护这样一个系统的必需工具。
它是一个专注于实时处理的任务队列，同时也支持任务调度。
Celery 有广泛、多样的用户与贡献者社区，你可以通过 IRC 或是 邮件列表 加入我们。
Celery 是开源的，使用 BSD 许可证 授权。

官网地址：http://docs.jinkan.org/docs/celery/

### 安装环境
> 服务器：Ubuntu 12.04.5 LTS (GNU/Linux 3.2.0-67-generic x86_64)

* 安装RabbitMQ
    * 安装RabbitMQ Server
        * `sudo apt-get install rabbitmq-server`
        * >RabbitMQ提供了一些简单实用的命令用于管理服务器运行状态：
        查看服务器运行状态: enable rabbitmq_management
        启动服务器:rabbitmq-server start
        停止服务器:rabbitmq-server stop
        查看服务器中所有的消息队列信息 :rabbitmqctl list_queues
        查看服务器种所有的路由信息: rabbitmqctl list_exchanges
        查看服务器种所有的路由与消息队列绑定信息 :rabbitmq list_bindings
    * 启用WEB管理台
    ```
        /usr/lib/rabbitmq/bin
        sudo ./rabbitmq-plugins enable rabbitmq_management
    ```
    * 添加远程管理账户
        将下面配置写入/etc/rabbitmq/rabbitmq.conf.d/rabbitmq.config文件中
        ```
            [
                {rabbit, [{tcp_listeners, [5672]}, {loopback_users, ["ken"]}]}
            ].
        ```
        执行命令
        ```
            cd /usr/lib/rabbitmq/bin/
            sudo rabbitmqctl add_user ken 123456
            sudo rabbitmqctl set_user_tags ken administrator
            sudo rabbitmqctl set_permissions -p / ken ".*" ".*" ".*"
        ```
* 安装Celery
    Celery详情查看[官方文档](http://docs.jinkan.org/docs/celery/getting-started/first-steps-with-celery.html)

    * 使用pip安装
    ```
        pip install Celery
    ```

### Celery方法示例
* 新建一个task.py

```
from celery import Celery

celery = Celery('tasks', broker='amqp://')
celery.conf.CELERY_RESULT_BACKEND = os.environ.get('CELERY_RESULT_BACKEND', 'amqp')

@celery.task(name='task.db_operation')
def db_operation(id):
    # 耗时的数据库操作
    pass
```

* 使用worker参数执行我们的程序的task

```
celery -A tasks worker --loglevel=info
```

* 新建一个handler.py

```
import tcelery
tcelery.setup_nonblocking_producer()

from tasks import db_operation

calss Resource(RequestHandler):
    @asynchronous
    def get():
        # 参数通过args传递,回调通过callback指定
        db_operation.apply_async(args=[id], callback=self.on_success)
    def on_success(self, response):
        # 获取返回的结果
        resource = response.result
        self.write(resource)
        self.finish()
```

此时，Resource的Get请求已经变成异步非阻塞了。


### run_on_executor方法示例

* 新建一个handler.py

```
from concurrent.futures import ThreadPoolExecutor
from tornado.concurrent import run_on_executor

class ChatHandler(web.RequestHandler):
    executor = ThreadPoolExecutor(4)

    @web.asynchronous
    @gen.coroutine
    def get(self):
        resource = yield self.get_db_operation()
        self.write(resource)
        self.finish()

    @web.asynchronous
    @gen.coroutine
    def post(self):
        yield self.post_db_operation()
        self.write('success')
        self.finish()

    @run_on_executor
    def get_db_operation(self):
        return resource

    @run_on_executor
    def post_db_operation(self):
        pass
```

### 总结
这一整套走下来，个人觉得使用Celery部署麻烦，而且一旦大量使用Celery，极有可能导致队列长度过长，影响处理效率。最后我选择使用了run_on_executor方法。
