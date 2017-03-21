## 序言

如果你非常熟悉前置知识, 再看workerman的源码就非常简单了.

## Workerman目录
```text
-----------Connection (对传输层的协议进行处理 TCP,UDP)
|
|----------Event (Reactor模型)
|
|----------Lib (一些工具类)
|
|----------Protocols  (对应用层协议进行处理 HTTP, websocket, text, 其它自定义协议)
|
|----------Worker.php (一个基本单元, 负责客户端连接的接收和释放)
|
|----------WebServer.php (继承自Worker, 应用层协议为http)

```
