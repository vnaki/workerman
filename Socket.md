## Socket编程

在`linux`系统中, 一切都是文件, `socket`也只是一个特殊的文件. 所有的文件都可以使用`read`, `write`进行操作.

### Socket server创建标准流程

- create
- bind
- listen

不过php提供一个非常方便的函数, `stream_socket_server`, 一个函数搞定上面三个步骤.

### PHP示例

```php
$fd = stream_socket_server("tcp://0.0.0.0:8090", $errno, $errstr);
```

> 上述代码就创建了一个socket.

```php
$conn = stream_socket_accept($fd);
```

> 上述代码调用`stream_socket_accept`, 会阻塞在这儿, 所谓阻塞, 就是等待事件的发生, 如果事件没有发生, 就会持续等待. 当有客户端有新请求发起时, 事件发生, 激活进程, 继续向下执行.

```php
fwrite($conn, "HTTP/1.0 200 OK\r\nContent-Type: 2\r\n\r\nHi");
fclose($conn);
```

> 在文章开头时说了, `socket`也是文件, 所以可以使用诸如`read`, `write`这样的函数.


### 综合示例

```php
<?php
$fd = stream_socket_server("tcp://0.0.0.0:8090", $errno, $errstr);

//省略错误处理

while (true) {
    $conn = stream_socket_accept($fd);

    $pid = pcntl_fork();
    if ($pid == 0) {
        $message = "Hi";
        $len = strlen($message);
        fwrite($conn, "HTTP/1.0 200 OK\r\nContent-Length: $len\r\n\r\n$message");
        fclose($conn);
        exit(0);
    } elseif ($pid > 0) {
        continue;
    } else {
        printf("fork failed");
    }
}
```
- 创建socket
- 调用`stream_socket_accept`, 进程阻塞在这里.
- 当有新请求发起时, fork一个子进程进行处理.
- 子进程处理完毕后, 结束进程, 释放资源.


> 上述代码是同步阻塞的方式, 高并发的时候效率低下, 现在最流行的方式是异步非阻塞.


异步非阻塞要配合`Event`来实现.

```php
stream_set_blocking($fd, 0);      //设置成非阻塞方式
```

```php
<?php

$fd = stream_socket_server("tcp://0.0.0.0:9001", $errno, $errstr);

stream_set_blocking($fd, 0);

$event_base = new EventBase();

$event = new Event($event_base, $fd, Event::READ | Event::PERSIST, function ($fd) use (&$event_base) {
 $conn = stream_socket_accept($fd);

 fwrite($conn, "HTTP/1.0 200 OK\r\nContent-Length: 2\r\n\r\nHi");
 fclose($conn);
}, $fd);

$event->add();

$event_base->loop();    
```

> 为了利用CPU多核的优势, 往往会fork出好几个进程同时监听一个端口. 可以通过`stream_context_set_option`来进行设置.

```php
stream_context_set_option($fd, 'socket', 'so_reuseport', 1);
```