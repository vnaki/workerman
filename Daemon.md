## 守护进程

为什么需要守护进程呢? 普通进程在运行时可以被控制终端发出的信号打断, 而守护进程由于脱离了控制终端, 所以不会被其打断.

## 守护进程创建的标准流程

- 调用umask设置文件创建的掩码
- fork子进程并关闭父进程
- 调用posix_setsid创建新会话
- 把当前工作目录切换为根目录
- 不需要的文件描述符全部关闭
- 标准输入输出错误重定向

除了第2步和第3步，其它的其实都是可选的. 

## 守护进程PHP代码

```php
function daemon()
{
    umask(0);
    $pid = pcntl_fork();
    if ($pid > 0) {
        exit(0);
    } elseif ($pid < 0) {
        printf("fork failed");
    }

    posix_setsid();

    $pid = pcntl_fork();
    if ($pid > 0) {
        exit(0);
    } elseif ($pid < 0) {
        printf("fork failed");
    }
}
```

把一个进程变成守护进程, 首先要让该进程脱离当前的控制终端, 要达到这个目的需要调用`posix_setsid`.

但我们的代码为什么在调用`posix_setsid`之前要`fork`一次呢? 这是因为调用`posix_setsid`的进程必须不能是`session leader`, 为了确保万无一失, 会先`fork`一个子进程, `fork`出来的子进程就必然不是`session leader`了. 

那在调用`posix_setsid`之后为什么还要再`fork`一次呢? 其实这必不是必须的, `nginx`在实现`daemon`时就没有`fork`两次.
很多`daemon`的实现都没有`fork`两次. 只是有人推荐在`sysv system`上, 再`fork`一次, 可以避免守护进程打开控制终端, 因为再`fork`一次之后, 子进程就不是`session leader`了.
