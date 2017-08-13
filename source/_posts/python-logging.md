---
title: Python的logging模块的复习和使用
date: 2017-08-13 11:02:22
tags: logging
category: Python
---

在之前的多个项目中都有用到过logging模块，虽然用过，但没有研究过，只是在每个代码文件中使用`logger=logging.getLogger(__name__)`这样简单粗暴的方法。

在这次的使用经历中，觉得logging模块真的很不错。

### logging
使用logging前，必须先要了解它的重要概念。

* **Logging Levels**
* **Logger Objects**
* **Handler Objects**
* **Formatter Objects**
* **Filter Objects**

### Level
Level 级别。只有在日志消息的级别大于logger和handler设定的级别，才会显示。

|级别|何时使用|
|----|-------|
|NOTSET|不设置级别|0|
|DEBUG|详细信息，调试时|10|
|INFO|证明事情按预期工作。|20|
|WARNING|表明发生了一些意外，或者不久的将来会发生问题（如‘磁盘满了’）。软件还是在正常工作。|30|
|ERROR|由于更严重的问题，软件已不能执行一些功能了。|40|
|CRITICAL|严重错误，表明软件已不能继续运行了。|50|

### Logger
Logger 记录器，暴露了应用程序代码能直接使用的接口。

Logger是一个树形层级结构，在使用接口debug，info，warn，error，critical之前必须创建Logger实例，即创建一个记录器，如果没有显式的进行创建，则默认创建一个root logger，并应用默认的日志级别(WARN)，处理器Handler(StreamHandler，即将日志信息打印输出在标准输出上)，和格式化器Formatter(默认的格式即为第一个简单使用程序中输出的格式)。

创建方法: `logger = logging.getLogger(name=None)`。如果不指定name则返回root logger对象。

常用方法如下：

|方法|说明|
|---------------------|---------------------------------------|
|addHandler(self, hdlr)|Add the specified handler to this logger.|
|removeHandler(self, hdlr)|Remove the specified handler from this logger.|
|setLevel(self, level)|Set the logging level of this logger.|


### Handler
Handler 处理器，将（记录器产生的）日志记录发送至合适的目的地。

比较常用的有三个，StreamHandler，FileHandler，NullHandler，详情可以访问[地址](!https://docs.python.org/2.7/library/logging.handlers.html#module-logging.handlers)

Handler处理器类型如下：

* StreamHandler
* FileHandler
* NullHandler
* WatchedFileHandler
* RotatingFileHandler
* TimedRotatingFileHandler
* SocketHandler
* DatagramHandler
* SysLogHandler
* NTEventLogHandler
* SMTPHandler
* MemoryHandler
* HTTPHandler

### Filter
过滤器，提供了更好的粒度控制，它可以决定输出哪些日志记录。


使用Formatter对象设置日志信息最后的规则、结构和内容，默认的时间格式为%Y-%m-%d %H:%M:%S。

创建方法: `formatter = logging.Formatter(fmt=None, datefmt=None)`

>其中，fmt是消息的格式化字符串，datefmt是日期字符串。如果不指明fmt，将使用'%(message)s'。如果不指明datefmt，将使用ISO8601日期格式。

### Formatter
格式化器，指明了最终输出中日志记录的布局。

Handlers和Loggers可以使用Filters来完成比级别更复杂的过滤。Filter基类只允许特定Logger层次以下的事件。例如用‘A.B’初始化的Filter允许Logger ‘A.B’, ‘A.B.C’, ‘A.B.C.D’, ‘A.B.D’等记录的事件，logger‘A.BB’, ‘B.A.B’ 等就不行。 如果用空字符串来初始化，所有的事件都接受。

|格式|描述|
|---|---|
|%(levelno)s|打印日志级别的数值|
|%(levelname)s|打印日志级别名称|
|%(pathname)s|打印当前执行程序的路径|
|%(filename)s|打印当前执行程序名称|
|%(funcName)s|打印日志的当前函数|
|%(lineno)d|打印日志的当前行号|
|%(asctime)s|打印日志的时间|
|%(thread)d|打印线程id|
|%(threadName)s|打印线程名称|
|%(process)d|打印进程ID|
|%(message)s|打印日志信息|

### 使用过程
在项目启动中导入logging模块，然后进行配置，logging标准模块支持三种配置方式: dictConfig，fileConfig，listen。其中，dictConfig是通过一个字典进行配置Logger，Handler，Filter，Formatter；fileConfig则是通过一个文件进行配置；而listen则监听一个网络端口，通过接收网络数据来进行配置。当然，除了以上集体化配置外，也可以直接调用Logger，Handler等对象中的方法在代码中来显式配置。

basicConfig关键字参数

|关键字|描述|
|------|----|
|filename|创建一个FileHandler，使用指定的文件名，而不是使用StreamHandler。|
|filemode|如果指明了文件名，指明打开文件的模式（如果没有指明filemode，默认为'a'）。|
|format|handler使用指明的格式化字符串。|
|datefmt|使用指明的日期／时间格式。|
|level|指明根logger的级别。|
|stream|使用指明的流来初始化StreamHandler。该参数与'filename'不兼容，如果两个都有，'stream'被忽略。|

例子：
logging.yaml
```
logging:
    version: 1
    formatters:
      brief:
        format: '%(message)s'
      default:
        format: '%(asctime)s %(levelname)-8s %(name)-15s %(message)s'
        datefmt: '%Y-%m-%d %H:%M:%S'
      fluent_fmt:
        '()': fluent.handler.FluentRecordFormatter
        format:
          loglevel: '%(levelname)s'
          hostname: '%(hostname)s'
          where: '%(module)s.%(funcName)s'
          stack_trace: '%(exc_text)s'
    handlers:
        console:
            class : logging.StreamHandler
            level: DEBUG
            formatter: default
            stream: ext://sys.stdout
        fluent_dev:
            class: fluent.handler.FluentHandler
            host: 127.0.0.1
            port: 24224
            tag: test.dev
            formatter: fluent_fmt
            level: DEBUG
        fluent_prod:
            class: fluent.handler.FluentHandler
            host: 127.0.0.1
            port: 24224
            tag: test.prod
            formatter: fluent_fmt
            level: DEBUG
        null:
            class: logging.NullHandler

    loggers:
        amqp:
            handlers: [null]
            propagate: False
        conf:
            handlers: [null]
            propagate: False
        dev:
            handlers: [console, fluent_dev]
            level: DEBUG
            propagate: False
        prod:
            handlers: [console, fluent_prod]
            level: DEBUG
            propagate: False
        '': # root logger
            handlers: [console]
            level: DEBUG
            propagate: False
```

导入配置方法：
```
import yaml
import logging
import logging.config

with open('logging.yaml') as fd:
    conf = yaml.load(fd)

logging.config.dictConfig(conf['logging'])

```

获取logger
```
logger = logging.getLogger(loggername)
```
