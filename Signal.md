## 信号

进程间通信常用的方法之一，就是发送信号.

## 发送信号

最常用的`linux`命令之一`kill`, 就是向进程发送一个信号. PHP中也有对应的函数`posix_kill`.

## 安装信号处理器

`pcntl_signal`, 安装一个信号处理器.

## 信号处理的两种方式

信号处理有两种方式, 一种是通过`declare ticks`方式, 一种是`pcntl_signal_dispatch`方式. 代码如下:

index.php
```php

//declare方式
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

/*

$flag = true;
pcntl_signal(SIGINT, function($signal) use (&$flag) {
    echo 'sigint', PHP_EOL;
    $flag = false;
});

file_put_contents("pid.txt", posix_getpid());

//some task
while ($flag) {
    pcntl_signal_dispatch();
}
 */

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

但其实这两种方式的本质是一样的, 都是通过`pcntl_signal_dispatch`来实现的:

`pcntl.c`源代码第512行~519行:

```c
PHP_MINIT_FUNCTION(pcntl)
{
    php_register_signal_constants(INIT_FUNC_ARGS_PASSTHRU);
    php_pcntl_register_errno_constants(INIT_FUNC_ARGS_PASSTHRU);
    php_add_tick_function(pcntl_signal_dispatch, NULL);

    return SUCCESS;
}
```

我们看到这一行代码`php_add_tick_function(pcntl_signal_dispatch, NULL);`

`register_tick_function`本质上也是调用`php_add_tick_function`, 源代码在`basic_function.c`的第5675行.
```c
PHP_FUNCTION(register_tick_function)
{
    user_tick_function_entry tick_fe;
    int i;
    zend_string *function_name = NULL;

    tick_fe.calling = 0;
    tick_fe.arg_count = ZEND_NUM_ARGS();

    if (tick_fe.arg_count < 1) {
        WRONG_PARAM_COUNT;
    }
    
    ......

    if (!BG(user_tick_functions)) {
        BG(user_tick_functions) = (zend_llist *) emalloc(sizeof(zend_llist));
        zend_llist_init(BG(user_tick_functions),
                        sizeof(user_tick_function_entry),
                        (llist_dtor_func_t) user_tick_function_dtor, 0);
        php_add_tick_function(run_user_tick_functions, NULL);
    }
    
    ......

    RETURN_TRUE;
}
```

所以`pcntl`扩展在模块初始化的时候就已经注册了一个`tick`函数. 相当于下面这句PHP代码:

```php
register_tick_function('pcntl_signal_dispatch');
```

这也验证了手册上的一句话:

> 在编译PHP时 启用pcntl将始终承担这种开销，不论您的脚本中是否真正使用了pcntl。

原因就是`pcntl`在模块初始化时就注册了这个函数.