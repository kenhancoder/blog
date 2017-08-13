---
title: Ubuntu中为Apache使用uWsgi
date: 2017-08-13 10:41:18
tags:
    - Apache
    - uWsgi
---

### 安装依赖

```shell
sudo apt-get install libapache2-mod-uwsgi
```

### 启动module
module具体名称需要进入/etc/apache2/mods-available/确认。在我的机子上为uwsgi.load
```shell
sudo a2enmod uwsgi
```

### Apache配置

```shell
<VirtualHost *:8001>
    <Location />
        SetHandler uwsgi-handler
        uWSGISocket 127.0.0.1:7001
    </Location>
</VirtualHost>
```

> * 8001：Apache监听端口
> * uWSGISocket：uWsgi启动时所使用的地址与端口
