---
title: 使用Fluentd、Elasticsearch、Kibana共同组件一个实时日志分析平台
date: 2017-08-13 10:58:27
tags:
    - Fluentd
    - Elasticsearch
    - Kibana
---

* Elasticsearch是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。

* Fluentd是一个日志收集系统，它的特点在于其各部分均是可定制化的，你可以通过简单的配置，将日志收集到不同的地方。

* Kibana也是一个开源和免费的工具，它Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志。


Fluentd的安装在博客中有提及到，在这里变不多讲了[地址](!http://hanliyi.ml/blog/share/fluentd_1.html)。Elasticsearch与Kibana的安装详情参考[地址](!http://www.tuicool.com/articles/QFvARfr)。

本文中主要是对Fluentd与Elasticsearch的对接进行说明，并创建Python脚本，为Python App来完成日志提交。

Fluentd配置文件
```
<match project.*>
  type copy
  <store>
    type stdout
    output_type json
  </store>
  <store>
    type elasticsearch
    host localhost
    port 9200
    index_name project
    type_name project
    flush_interval 5s
    logstash_format true
    logstash_prefix project
 </store>
</match>
```
我对match是`project.*`的日志分别输出到stdout和elasticsearch中。不知道为啥`index_name`与elasticsearch中的`_index`不一样。反而logstash_prefix是`_index`的前缀，并与当前日期拼接成为完整的`_index`。不知道是否和logstash_format设为true有关，但是如果没有logstash_format这个参数，elasticsearch就接收不到Fluentd的日志。


Python脚本
```Python
# coding: utf-8
import os
import logging
from fluent import handler

custom_format = {
    'host': '%(hostname)s',
    'where': '%(module)s.%(funcName)s',  # 具体到文件、函数
    'loglevel': '%(levelname)s',
    'stack_trace': '%(exc_text)s'
}

logging.basicConfig(level=logging.ERROR)

env = os.environ.get('GOOCANIMSERVICE_ENV') if os.environ.get(
    'GOOCANIMSERVICE_ENV') else 'local'
l = logging.getLogger('goocanim.' + env)

h = handler.FluentHandler('goocanim.' + env, host='121.40.99.27', port=24224)
formatter = handler.FluentRecordFormatter(custom_format)
h.setFormatter(formatter)

l.addHandler(h)
```

需要安装fluent-logger，安装命令`pip install fluent-logger`。在相应的文件中引用`l`，使用`l.info(...)`、`l.error(...)`等命令，就能将相应的日志输出到Elasticsearch中，并在Kibana中显示。
