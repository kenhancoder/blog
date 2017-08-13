---
title: 使用GEM方式安装Fluentd以及Fluentd-UI
date: 2017-08-13 10:57:52
tags: Fluentd
---

##### Fluentd
Fluentd是一个日志收集系统，它的特点在于其各部分均是可定制化的，你可以通过简单的配置，将日志收集到不同的地方。

##### Fluentd-UI
FLuentd-UI提供了一个图形界面，用于对Fluentd的管理。可以对Fluentd进行启动、停止、配置等等操作

##### Fluentd安装
系统：Ubuntu 14.04.5 LTS

使用了Github[地址](!https://github.com/fluent/fluentd/)上的安装方式----gem。

虽然安装命令简单`gem install fluentd`，但是安装得好慢、好慢、好慢。。。。

因为是使用gem安装的，所以需要Ruby。官方文档要求Ruby>=1.9.3，我在机器上安装了Ruby2.3。

在`/etc`中创建`fluent`目录，在目录中运行`fluentd -s conf`初始化配置，此时会创建一个conf目录，里面是fluentd的各种配置文件。运行`sudo fluentd -c conf/fluent.conf`启动fluentd。此时在新的Bash中运行`echo '{"json":"message"}' | fluent-cat debug.test`，然后在运行fluentd的Bash中将会见到`debug.test: {"json":"message"}`出现，这样fluentd就安装好了。

##### Fluentd-UI安装
与Fluentd相同，选择gem方式安装，命令是`gem install -V fluentd-ui`

因为Fluentd-UI可以对Fluentd进行管理，所以我便没有在机器上使用`fluentd -s conf`、`sudo fluentd -c conf/fluent.conf`来初始配置以及运行。

Usages:
```
fluentd-ui help [COMMAND]  # Describe available commands or one specific command
fluentd-ui setup           # setup fluentd-ui server
fluentd-ui start           # start fluentd-ui server
fluentd-ui status          # status of fluentd-ui server
fluentd-ui stop            # stop fluentd-ui server
```

使用了Supervisor来启动Fluentd-UI。
```
[program:fluentd_ui]
command=fluentd-ui start
```
