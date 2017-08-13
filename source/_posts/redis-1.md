---
title: Python中使用Redis遭遇打开文件数过多
date: 2017-08-13 11:03:45
tags: Redis
---

今天服务突然爆出好多异常日志：
`ConnectionError: Error 24 connecting to 127.0.0.1:6379. Too many open files.`

通过命令`cat /proc/(redis-sever_PID)/limits`获得
```
Max open files  4016  4016  files
```

第一反应是觉得可能4016过小，所以需要对其升值。

编辑/etc/sysctl.conf 添加如下内容：
```
fs.file-max = 65535
```
运行`sysctl -p`

再编辑 /etc/security/limits.conf:
```
root soft nofile 65536
root hard nofile 65536
* soft nofile 65536
* hard nofile 65536
```

使用命令`ulimit -n 65535`，将最大文件数提高至65535，这种修改只是在当前shell中生效，想要真正对Redis中**Max open files**进行修改，就需要在当前shell中进行重启redis，然后服务就恢复了正常。**这种方法其实是治标不治本的**

在临时解决异常问题后，需要思考为什么会有这么多未释放的连接。

经过分析，在创建Redis连接时，都会创建一个ConnectionPool。这导致，每获取一个StrictRedis时，都会创建一个新的ConnectionPool，这才是导致文件数过多的原因。

解决方案：

先创建创建一个ConnectionPool
`pool = redis.ConnectionPool(host='localhost', port=6379, db=0)`

然后创建一个Redis
`r = redis.Redis(connection_pool=pool)`
