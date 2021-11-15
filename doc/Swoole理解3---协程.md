## Swoole理解3---协程

Swoole已经支持协程，不用你就落后了！

内容概览:
```
协程的执行顺序: 初窥 swoole 中协程的调度
协程为什么快: 减少IO阻塞带来的性能损耗
swoole 协程和 go 协程对比: 单进程 vs 多线程
```

协程的执行顺序
先来看看基础的例子:
```
go(function () {
    echo "hello go1 \n";
});

echo "hello main \n";

go(function () {
    echo "hello go2 \n";
});
```

go() 是 \Co::create() 的缩写, 用来创建一个协程, 接受 callback 作为参数, callback 中的代码, 会在这个新建的协程中执行.

备注: \Swoole\Coroutine 可以简写为 \Co

上面的代码执行结果:

```
root@b98940b00a9b /v/w/c/p/swoole# php co.php
hello go1
hello main
hello go2
```


执行结果和我们平时写代码的顺序, 好像没啥区别. 实际执行过程:

运行此段代码, 系统启动一个新进程
遇到 go(), 当前进程中生成一个协程, 协程中输出 heelo go1, 协程退出
进程继续向下执行代码, 输出 hello main
再生成一个协程, 协程中输出 heelo go2, 协程退出
运行此段代码, 系统启动一个新进程. 如果不理解这句话, 你可以使用如下代码:
```
// co.php
<?php

sleep(100);
```
执行并使用 ps aux 查看系统中的进程:
```
root@b98940b00a9b /v/w/c/p/swoole# php co.php &
⏎
root@b98940b00a9b /v/w/c/p/swoole# ps aux
PID   USER     TIME   COMMAND
    1 root       0:00 php -a
   10 root       0:00 sh
   19 root       0:01 fish
  749 root       0:00 php co.php
  760 root       0:00 ps aux
⏎
```
我们来稍微改一改, 体验协程的调度:
```
use Co;

go(function () {
    Co::sleep(1); // 只新增了一行代码
    echo "hello go1 \n";
});

echo "hello main \n";

go(function () {
    echo "hello go2 \n";
});
```
\Co::sleep() 函数功能和 sleep() 差不多, 但是它模拟的是 IO等待(IO后面会细讲). 执行的结果如下:
```
root@b98940b00a9b /v/w/c/p/swoole# php co.php
hello main
hello go2
hello go1
```


怎么不是顺序执行的呢? 实际执行过程:

运行此段代码, 系统启动一个新进程
遇到 go(), 当前进程中生成一个协程
协程中遇到 IO阻塞 (这里是 Co::sleep() 模拟出的 IO等待), 协程让出控制, 进入协程调度队列
进程继续向下执行, 输出 hello main
执行下一个协程, 输出 hello go2
之前的协程准备就绪, 继续执行, 输出 hello go1
到这里, 已经可以看到 swoole 中 协程与进程的关系, 以及 协程的调度, 我们再改一改刚才的程序:
```
go(function () {
    Co::sleep(1);
    echo "hello go1 \n";
});

echo "hello main \n";

go(function () {
    Co::sleep(1);
    echo "hello go2 \n";
});
```
我想你已经知道输出是什么样子了:
```
root@b98940b00a9b /v/w/c/p/swoole# php co.php
hello main
hello go1
hello go2
```

协程快在哪? 减少IO阻塞导致的性能损失
大家可能听到使用协程的最多的理由, 可能就是 协程快. 那看起来和平时写得差不多的代码, 为什么就要快一些呢? 一个常见的理由是, 可以创建很多个协程来执行任务, 所以快. 这种说法是对的, 不过还停留在表面.

首先, 一般的计算机任务分为 2 种:

CPU密集型, 比如加减乘除等科学计算
IO 密集型, 比如网络请求, 文件读写等
其次, 高性能相关的 2 个概念:

并行: 同一个时刻, 同一个 CPU 只能执行同一个任务, 要同时执行多个任务, 就需要有多个 CPU 才行
并发: 由于 CPU 切换任务非常快, 快到人类可以感知的极限, 就会有很多任务 同时执行 的错觉
了解了这些, 我们再来看协程, 协程适合的是 IO 密集型 应用, 因为协程在 IO阻塞 时会自动调度, 减少IO阻塞导致的时间损失.

我们可以对比下面三段代码:

普通版: 执行 4 个任务
```
$n = 4;
for ($i = 0; $i < $n; $i++) {
    sleep(1);
    echo microtime(true) . ": hello $i \n";
};
echo "hello main \n";
```
```
root@b98940b00a9b /v/w/c/p/swoole# time php co.php
1528965075.4608: hello 0
1528965076.461: hello 1
1528965077.4613: hello 2
1528965078.4616: hello 3
hello main
real    0m 4.02s
user    0m 0.01s
sys     0m 0.00s
```
单个协程版:
```
$n = 4;
go(function () use ($n) {
    for ($i = 0; $i < $n; $i++) {
        Co::sleep(1);
        echo microtime(true) . ": hello $i \n";
    };
});
echo "hello main \n";
```
```
root@b98940b00a9b /v/w/c/p/swoole# time php co.php
hello main
1528965150.4834: hello 0
1528965151.4846: hello 1
1528965152.4859: hello 2
1528965153.4872: hello 3
real    0m 4.03s
user    0m 0.00s
sys     0m 0.02s
```
多协程版: 见证奇迹的时刻
```
$n = 4;
for ($i = 0; $i < $n; $i++) {
    go(function () use ($i) {
        Co::sleep(1);
        echo microtime(true) . ": hello $i \n";
    });
};
echo "hello main \n";
```

```
root@b98940b00a9b /v/w/c/p/swoole# time php co.php
hello main
1528965245.5491: hello 0
1528965245.5498: hello 3
1528965245.5502: hello 2
1528965245.5506: hello 1
real    0m 1.02s
user    0m 0.01s
sys     0m 0.00s
⏎
```
为什么时间有这么大的差异呢:

普通写法, 会遇到 IO阻塞 导致的性能损失
单协程: 尽管 IO阻塞 引发了协程调度, 但当前只有一个协程, 调度之后还是执行当前协程
多协程: 真正发挥出了协程的优势, 遇到 IO阻塞 时发生调度, IO就绪时恢复运行
我们将多协程版稍微修改一下:

多协程版2: CPU密集型
```
$n = 4;
for ($i = 0; $i < $n; $i++) {
    go(function () use ($i) {
        // Co::sleep(1);
        sleep(1);
        echo microtime(true) . ": hello $i \n";
    });
};
echo "hello main \n";
```

```
root@b98940b00a9b /v/w/c/p/swoole# time php co.php
1528965743.4327: hello 0
1528965744.4331: hello 1
1528965745.4337: hello 2
1528965746.4342: hello 3
hello main
real    0m 4.02s
user    0m 0.01s
sys     0m 0.00s
```
只是将 Co::sleep() 改成了 sleep(), 时间又和普通版差不多了. 因为:

sleep() 可以看做是 CPU密集型任务, 不会引起协程的调度
Co::sleep() 模拟的是 IO密集型任务, 会引发协程的调度
这也是为什么, 协程适合 IO密集型 的应用.

再来一组对比的例子: 使用 redis



```
// 同步版, redis使用时会有 IO 阻塞
$cnt = 2000;
for ($i = 0; $i < $cnt; $i++) {
    $redis = new \Redis();
    $redis->connect('redis');
    $redis->auth('123');
    $key = $redis->get('key');
}

// 单协程版: 只有一个协程, 并没有使用到协程调度减少 IO 阻塞
go(function () use ($cnt) {
    for ($i = 0; $i < $cnt; $i++) {
        $redis = new Co\Redis();
        $redis->connect('redis', 6379);
        $redis->auth('123');
        $redis->get('key');
    }
});

// 多协程版, 真正使用到协程调度带来的 IO 阻塞时的调度
for ($i = 0; $i < $cnt; $i++) {
    go(function () {
        $redis = new Co\Redis();
        $redis->connect('redis', 6379);
        $redis->auth('123');
        $redis->get('key');
    });
}

```
性能对比:
```
# 多协程版
root@0124f915c976 /v/w/c/p/swoole# time php co.php
real    0m 0.54s
user    0m 0.04s
sys     0m 0.23s
⏎

# 同步版
root@0124f915c976 /v/w/c/p/swoole# time php co.php
real    0m 1.48s
user    0m 0.17s
sys     0m 0.57s
⏎
```

