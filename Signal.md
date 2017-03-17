## 信号

进程间通信常用的方法之一，就是发送信号.

## 发送信号

最常用的`linux`命令之一`kill`, 就是向进程发送一个信号. PHP中也有对应的函数`posix_kill`.

## 安装信号处理器

`pcntl_signal`, 安装一个信号处理器.

## PHP示例代码

index.php
```php

declare(ticks=1);

$flag = true;
pcntl_signal(SIGINT, function($signal) use (&$flag) {
    echo 'sigint', PHP_EOL;
    $flag = false;
});

file_put_contents("pid.txt", posix_getpid());

//some task
while ($flag) {
    usleep(100);
}
```

```shell
php index.php &
```

sendSignal.php
```php
posix_kill(file_get_contents("pid.txt"), SIGINT);
```

```shell
php sendSignal.php
```