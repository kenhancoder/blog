---
title: 使用Tornado中的WebSocket
date: 2017-08-13 11:06:09
tags: 
    - Tornado
    - WebSocket
---

Tornado已经实现了对WebSocket的封装。

以下是源码提供Demo的部分代码。Tornado的github地址：<https://github.com/tornadoweb/tornado>

```python
class ChatSocketHandler(tornado.websocket.WebSocketHandler):
    waiters = set()

    def open(self):
        ChatSocketHandler.waiters.add(self)

    def on_close(self):
        ChatSocketHandler.waiters.remove(self)

    def on_message(self, message):
        logging.info("got message %r", message)
        self.write_message(u"You said: " + message)
```
在此ChatSocketHandler中override了open、on_close、on_message方法。
> * open: 在此方法体内，可以进行开启连接后的操作
> * on_close: 在此方法体内，可以进行关闭连接后的操作
> * on_message: 在此方法体内，可以对传入的消息进行操作
> * 使用write_message方法向已连接客户端发送消息

如果仅仅使用以上的方法，在实际开发中将会遇到跨域的问题。这时需要override下WebSocketHandler中的check_origin

```python
def check_origin(self, origin):
    return True
```
