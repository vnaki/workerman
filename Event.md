## Libevent

libevent(这里的libevent是一个库)封装了`linux`底层的`epoll`, 安装`Libevent`(这里的libevent是一个PHP扩展, 用来连接PHP与库)扩展或者`Event`扩展让php可以调用`libevent`封装好的函数. 支持I/O, 定时器, 信号. 

## php类
- `Event`
- `EventBase`

## 示例

### I/O示例
```php


<?php

$fd = stream_socket_server("tcp://0.0.0.0:9001", $errno, $errstr);

stream_set_blocking($fd, 0);

$event_base = new EventBase();

$event = new Event($event_base, $fd, Event::READ | Event::PERSIST, function ($fd) use (&$event_base) {
 $conn = stream_socket_accept($fd);

 fwrite($conn, "HTTP/1.0 200 OK\r\nContent-Length: 3\r\n\r\nHi");
 fclose($conn);
}, $fd);

$event->add();

$event_base->loop();                                                                                                         

```

### 定时器示例

```php
<?php

function setTimeout($second, $callback)
{
    $event_base = new EventBase();

    $event = Event::timer($event_base, $callback);

    $event->add($second);

    $event_base->loop();
}

function setInterval($second, $callback)
{
    $func = function() use ($second, $callback, &$func) {
        call_user_func($callback);
        setTimeout($second, $func);
    };

    setTimeout($second, $func);
}


setInterval(1, function() {
    echo "hello", PHP_EOL;
});
```

### 信号示例
```php
<?php 

$pid = pcntl_fork();

if ($pid > 0) {
    sleep(1);
    posix_kill($pid, SIGINT);
    exit(0);

} elseif ($pid == 0) {
    $event_base = new EventBase();

    $event = Event::signal($event_base, SIGINT, function() use (&$event, &$event_base) {
        echo "sigint", PHP_EOL;
        $event->del();
        $event_base->free();
        exit(0);
    });

    $event->add();

    $event_base->loop();

} else {
    printf("fork failed");
}
```