## Swoole理解4--Process进程

[官方文档] https://wiki.swoole.com/wiki/page/p-process.html

###Process简介
Swoole除了在Server中提供了固有的Worker进程和TaskWorker进程之外，还提供了一个Process模块用于动态创建新的进程。与PHP的pcntl进程模块相比，Swoole的Process提供了基于UnixSock的进程间通信方式，并且支持重定向标准输入输出到管道。

###Process使用
创建一个Process的函数原型和参数说明如下：
```
swoole_process::__construct(callable $function, $redirect_stdin_stdout = false, $create_pipe = 1);
```
|参数名|	参数解释 |
|:----    |-----   |
|function |	进程需要执行的函数|
|redirect_stdin_stdout	|是否需要重定向标准输入输出，默认为否|
|create_pipe	|创建管道的类型，1为流式，2为数据包模式，0代表不创建|

$function是进程启动后执行的逻辑内容。这里需要注意，如果这个函数执行完，那么整个进程会退出并被自动销毁。所以如果你希望创建一个常驻进程并且一直执行任务，那么就需要在$function中添加一个死循环（或者Swoole的事件循环），由此来保证进程不会退出。

$redirect_stdin_stdout选项如果开启，Process会默认创建管道，并且将标准输入STDIN重定向为读取管道，将标准输出STDOUT重定向为写入管道。这个选项是用于配合Process的exec方法使用，具体使用场景将在后面的小节中介绍

$create_pipe 用于指定是否创建管道以及管道的类型。如果选择流式管道，那么在管道中的数据是像TCP协议一样，不会自动拆包，需要在业务层实现分包逻辑；如果选择数据包模式，管道中发送的数据都会是一个个完整的数据包，读取的时候也不会出现一次读取多个数据的情况。


###进程间通信
####管道
上面提到过，Process进程提供了内置管道的方式用于进程间通信。在构建Process实例的时候，只要开启了create_pipe选项，Swoole底层就会自动创建一个管道（这里需要说明，虽然名字叫管道，但是实际上在新版本Swoole当中，实际的底层通信是通过Unix Socket实现的，而不是真正意义上的Linux Pipe）。

当进程被实际fork出来后，父进程和子进程中的Process对象会被设置一个名为pipe的成员变量，存放着底层Unix Socket的描述符，父进程和子进程可以通过这个描述符来发送数据，也可以直接调用Process提供的read/write接口来收发数据。示例如下：
```
function runner(swoole_process $sub_process)
{
    // 读取主进程发来的消息
    echo $sub_process->read();
    // 发送消息给主进程
    $sub_process->write("Hello Main Process\n");
    $sub_process->exit(0);
}

$process = new swoole_process("runner", false, 1);
$process->start();
// 发送消息给子进程
$process->write("Hello Sub Process\n");
// 读取子进程发来的消息
echo $process->read();
// 调用wait方法回收子进程
swoole_process::wait(true);
```
以上代码运行的输出为：
```
$ php process_pipe.php
Hello Sub Process
Hello Main Process
```

####消息队列
除了管道通信，Swoole的Process还支持使用系统自带的消息队列来实现进程间通信。

Process提供了用于开启和关闭消息队列的接口useQueue和freeQueue，这两个接口的原型如下：
```
bool swoole_process::useQueue(int $msgkey = 0, int $mode = 2);
void swoole_process::freeQueue();
```

其中，useQueue的第一个参数msgkey是消息队列的键值，在系统中，使用同一个key创建的队列指向同一个实例，因此我们可以使用同一个key来让所有的子进程共享同一个消息队列，也可以使用不同的key使得各个子进程使用独立的消息队列。

第二个参数mode有两种模式，1和2。

如果使用模式1，那么使用指定子进程对象推送消息，该消息就会在指定的子进程中收到。比如我们创建了10个Process对象，并编号为1-10，然后使用$process_3->push("")发送消息的话，这条消息就会在第三个子进程里收到，可以在进程回调中，通过$process->pop()来获取这条消息（即使使用同一个key创建消息队列， 也会如此）。这个模式适用于需要通知特定子进程的时候使用。
```
/**
 * @param swoole_process $sub_process
 */
function runner(swoole_process $sub_process)
{
    echo $sub_process->pop();
    $sub_process->push("Hello Main Process\n");
    $sub_process->exit(0);
}

$process = new swoole_process("runner", false, 1);
// 使用模式1
$process->useQueue(ftok(__FILE__, 'p'), 1);
$process->start();

$process->push("Hello Sub Process\n");
// 防止消息被父进程自己消费
sleep(1);   
echo $process->pop();
swoole_process::wait(true);
```
如果使用模式2，那么所有共用同一个消息队列的子进程会使用抢占模式争抢消息队列中的消息。还是用上面的10个子进程的例子，这10个子进程都在调用pop方法等待消息。如果这时使用任意一个子进程对象push一条消息进去，只会有其中某一个子进程抢到这个消息，其他子进程不会收到。这个模式适用于多进程消费模型，我们可以创建一个多个Process用于处理逻辑，然后使用消息队列来发逻辑，这样发送出去的任务会被自动被闲置的进程获取并执行。
```
$process = [];

for($i = 0; $i < 3; $i ++){
    $process[$i] = new swoole_process(function(swoole_process $sub_process) use ($i){
        while(true)
        {
            $data = $sub_process->pop();
            if( $data == 'exit' )
            {
                $sub_process->exit(0);
                break;
            }
            echo $i . ": " .$data;
            sleep(1);
        }
    }, false, 1);
    $process[$i]->useQueue(ftok(__FILE__, 'p'), 2);
    $process[$i]->start();
}
// 发送消息
$process[0]->push("Hello Sub Process\n");
$process[0]->push("Hello Sub Process\n");
$process[0]->push("Hello Sub Process\n");
$process[0]->push("Hello Sub Process\n");
// 通知子进程关闭
$process[0]->push("exit");
$process[0]->push("exit");
$process[0]->push("exit");
sleep(1);
swoole_process::wait(true);
swoole_process::wait(true);
swoole_process::wait(true);
```

以上代码就实现了一个抢占模式的多进程模型。该段程序的输出结果是
```
0: Hello Sub Process
1: Hello Sub Process
2: Hello Sub Process
0: Hello Sub Process
```
如果将子进程里的sleep(1)注释掉，则运行结果会变成:
```
0: Hello Sub Process
0: Hello Sub Process
0: Hello Sub Process
0: Hello Sub Process
```
这就可以看出，如果没有sleep将0号子进程标记为繁忙，该进程会抢占全部的4条消息。当设置之后，其他的进程也就能抢到消息了。


###使用实例
在实际使用Process当中，还有一些其他需要注意的事项。这里我会列举一些Process常用的API和使用场景，供大家在实际使用中选择

###常用API
Process提供了一些比较常用的API用于操作和设置进程，这些API如下列表所示：

name name方法用于设置进程的进程名，类似于PHP的cli_set_process_title方法。函数原型如下：
```
bool swoole_process::name(string $new_process_name);
```
函数执行成功后会返回true。需要注意的是，这个函数必须在子进程的回调函数中执行才能生效，在start前调用是不会生效的。

daemon daemon方法用于将子进程脱变为一个守护进程（即使退出终端也会在后台持续运行的进程）。函数原型如下：
// Swoole 1.9.1以前的版本，两个参数的默认值均为false
```
bool swoole_process::daemon(bool $nochdir = true, bool $noclose = true);
```
其中，nochdir参数为true表示不需要切换当前目录到根目录（守护进程默认会把当前工作目录切换为根目录）。noclose参数为true时表示不要关闭标准输入输出，这样即使脱变为守护进程，进程仍然可以接收来自管道的标准输入，并且向管道或者标准输出打印内容。

setaffinity setaffinity方法用于将进程绑定到指定的CPU核上，这可以让进程只在某些指定的CPU核上运行，从而空出部分CPU资源执行更重要的程序（或者将某些重要的进程指定在某个空余的CPU上运行）。函数原型如下：
```
bool swoole_process::setaffinity(array $cpu_set);
```
cpu_set为指定的CPU核的id，从0开始编号，最大值为CPU核数减1。

信号
在Linux环境中，我们往往需要通过kill命令给进程发送指定的信号来通知进程做一些事情，比如kill -SIGTERM会通知进程关闭自己。而进程本身也往往会注册对应的信号监听函数，用于在收到特定信号之后执行对应的操作。因此，Swoole的Process也提供了配套的API用于发送和接收信号。

Process提供了同名的kill方法用于向指定进程发送信号，其函数原型为：
```
int swoole_process::kill($pid, $signo = SIGTERM);
```
第一个参数pid是指定的进程号，第二个参数signo是需要发送的信号，默认为SIGTERM（程序终止）。这里如果指定signo为0的话，可以用该方法检测进程是否存在，底层并不会发送实际的信号。
```
if( swoole_process::kill($pid, 0) )
{
    echo "process exist";
}
```
当一个子进程退出之后，它会向自己的父进程发送一个SIGCHLD信号。这个时候如果父进程不进程对应的回收处理，这个子进程就会变成一个僵尸进程(Zombie Process)，如果产生大量的僵尸进程，会浪费操作系统的资源。因此，父进程需要监听SIGCHLD信号，并进行对应的回收处理操作。 Process提供了signal方法用于注册信号监听，其函数原型如下：
```
bool swoole_process::signal(int $signo, mixed $callback);
```
第一个参数signo是需要监听的信号，第二个参数callback是收到信号后执行的回调函数。需要注意的是，Process的signal方法是一个异步方法，其底层会开启事件循环，因此使用了signal方法的进程在主代码执行完后并不会主动退出，需要调用exit、发送信号等方式关闭。 当signal方法收到来自子进程的SIGCHLD信号后，需要执行Process的wait方法回收子进程。wait方法的函数原型如下：
```
array swoole_process::wait(bool $blocking = true);
```
参数blocking用于指定该函数是否阻塞等待。如果指定为true，那么wait函数会一直阻塞等待某个子进程退出；如果指定为false，那么调用wait方法的时候，如果当前状态没有子进程退出，该函数会立即返回，并且返回值为空。如果此时wait函数确实回收到了某个退出的子进程，其返回数组的结构如下：
```
array(
    'code' => 0,        // 退出状态码
    'pid' => 15001,     // 子进程号
    'signal' => 15      // 被哪种信号kill
);
```
因此，当我们需要在主进程中回收多个子进程的时候，可以使用如下代码来实现：
```
// 注册SIGCHLD信号
swoole_process::signal(SIGCHLD, function($sig) {
  // 回收子进程
  while($ret =  swoole_process::wait(false));
});
```
之所以使用循环来回收子进程，是因为在收到信号的时候，可能同一时刻有多个子进程退出，因此要用循环把所有的子进程都回收，直到wait方法返回false，此时代表所有退出的子进程都已回收完毕。


###定时器
前面我们提到过，Swoole本身提供了毫秒级精度的定时器tick和after，这两个定时器同样可以在Process中使用：

```
function runner(swoole_process $sub_process)
{
    swoole_timer_tick(1000, function(){
        echo "Hello World";
    });
}
$process = new swoole_process("runner", false, 1);
$process->start();
```

除了Swoole内置的定时器之外，在1.8.13及更高的Swoole版本中，Process还提供了一个精度更高的定时器。Linux操作系统提供了一个setitimer系统调用用于设置微妙级的定时器，该定时器会在指定的时间间隔向进程发送对应的信号。Process将这个系统调用封装为alarm方法，其原型如下：
```
bool swoole_process::alarm(int $interval_usec, int $type = 0);
```
|参数名|	参数说明|
|:---|-----|
|interval_usec	|定时器间隔时间，单位为微妙。如果为负数表示清除定时器。|
|type	|定时器类型 0 表示为真实时间,触发SIGALAM信号 1 表示用户态CPU时间，触发SIGVTALAM信号 2 表示用户态+内核态时间，触发SIGPROF信号|

因为定时器底层是发送信号给进程，因此进程必须使用signal方法监听信号，具体的示例代码如下：
```
swoole_process::signal(SIGALRM, function () {
    echo "usec timer done";
});

//100ms
swoole_process::alarm(100 * 1000);
```
另外，alarm定时器和Swoole本身的Timer定时器不能混用。

###运行外部程序
在PHP开发中，我们或多或少会遇到这样的需求：执行一个系统命令，或者调用一个外部脚本（shell脚本，Python脚本甚至某个PHP脚本等等）。当然，PHP语言本身就提供了足够多种类的API来完成这样的工作：exec，shell_exec，passthru等等。运行一些简短、耗时低、不需要依赖后续输入的命令或脚本，以上命令基本可以胜任，但是面对一些需要依赖交互的命令，例如gdb、top或者一些有持续性输出的脚本，这些命令就不太方便使用了。为此，Swoole的Process提供了exec命令，用于执行这一类特殊的命令。

Process的exec函数原型如下：
```
bool swoole_process->exec(string $execfile, array $args)
```
|参数名|	参数说明|
|:----|-----|
|execfile|	指定需要执行的命令的绝对路径，例如"/bin/sh"或者"/usr/bin/python"|
|args	|参数数组，是执行execfile指定命令时带入的参数，数组内参数顺序即为调用时的排列顺序|
例如，我们可以用如下的代码来调用一个外部的shell脚本：
```
$process->exec("/bin/sh", [
    '/root/test.sh',
    '-c100',
]);
这段代码等价于执行：

/bin/sh /root/test.sh -c100
```
当调用exec方法成功后，当前子进程的运行代码就会被替换为新的脚步，但是当前子进程仍然和父进程保持父子关系，并且两个进程直接可以通过标准输入输出进行通信（因此必须启用重定向标准输入输出）。

这里我们用一个例子来演示exec的用法。首先我们创建一个名为test.php的脚步，这个脚本接收一个参数。
```
// 打印命令行参数
echo $argv[1]. PHP_EOL;
// 读取标准输入
$msg = trim(fgets(STDIN));
// 打印输入的内容
echo "Hello ". $msg . PHP_EOL;
```
接着我们创建一个php文件用于执行Process：
```
function runner(swoole_process $sub_process)
{
    // 执行脚本
    $sub_process->exec("/usr/local/php7/bin/php", [
        __DIR__ . '/test.php',
        'hello'
    ]);
}

$process = new swoole_process("runner", true, 2);
$process->start();

// 读取输出
echo $process->read();
// 传入参数
$process->write("World\n");
// 读取输出
echo $process->read();
swoole_process::wait(true);
```
以上程序的运行输出结果为：
```
hello
Hello World
```

可以看到，子进程将父进程通过管道传递过来的参数，从自己的标准输入STDIN中读取到，并且打印到标准输出里的内容，也通过管道被父进程收到。这样，我们就能实现父进程和子进程之间的通信里。这里如果子进程没有开启重定向标准输入输出，那么父进程就不能和子进程通信。

###进程池
之前的例子，我们都只创建了一个子进程。而实际使用中，我们往往需要创建多个子进程来满足需求。因此，我们需要一个可管理的进程池来辅助我们管理和操作这些子进程对象。这里，我用一个例子来进行示范。

我们的目标是创建一个可以动态扩容的进程池，该进程池默认会创建3个进程来处理任务，当发现进程不够用的时候，会自动创建新的子进程。任务结束之后，所有的子进程都会被回收。
```
class BaseProcess
{
    private $process_list = [];         // 进程池对象数组
    private $process_use = [];          // 进程占用标记数组
    private $min_worker_num = 3;        // 进程池最小值
    private $max_worker_num = 6;        // 进程池最大值

    private $current_num;               // 当前进程数

    public function __construct()
    {
        $this->run();
    }

    public function run()
    {
        $this->current_num = $this->min_worker_num;
        
        // 初始化进程池
        for($i=0;$i<$this->current_num ; $i++){
            $process = new swoole_process(array($this, 'task_run') , false , 2);
            $pid = $process->start();
            $this->process_list[$pid] = $process;
            $this->process_use[$pid] = 0;
        }
        // 绑定子进程管道的读事件，接收子进程任务结束的通知
        foreach ($this->process_list as $process) {
            swoole_event_add($process->pipe, function ($pipe) use($process) {
                $data = $process->read();
                var_dump($data);
                $this->process_use[$data] = 0;
            });
        }
        // 使用定时器，每500ms派发一个任务，共发送10个
        swoole_timer_tick(500, function($timer_id) {
            static $index = 0;
            $index = $index + 1;
            $flag = true;
            // 查找是否有可用的进程派发任务
            foreach ($this->process_use as $pid => $used) {
                // 找到了闲置的进程
                if($used == 0)
                {
                    $flag = false;
                    $this->process_use[$pid] = 1;
                    // 派发任务
                    $this->process_list[$pid]->write($index . "Hello");
                    break;
                }
            }
            // 没有找到进程，并且进程池并没有满
            if( $flag && $this->current_num < $this->max_worker_num )
            {
                // 创建新的进程
                $process = new swoole_process(array($this, 'task_run') , false , 2);
                $pid = $process->start();
                $this->process_list[$pid] = $process;
                $this->process_use[$pid] = 1;
                // 派发任务
                $this->process_list[$pid]->write($index . "Hello");
                $this->current_num ++;
                // 绑定子进程管道的读事件
                swoole_event_add($process->pipe, function ($pipe) use($process) {
                    $data = $process->read();
                    var_dump($data);
                    $this->process_use[$data] = 0;
                });
            }
            var_dump($index);
            if( $index == 10 )
            {
                // 任务结束，退出所有子进程
                foreach ($this->process_list as $process) {
                    $process->write("exit");
                }
                swoole_timer_clear($timer_id);
            }
        });
    }

    public function task_run($worker)
    {
        // 注册监听管道的事件，接收任务
        swoole_event_add($worker->pipe, function ($pipe) use ($worker){
            $data = $worker->read();
            var_dump($worker->pid . ": " . $data);
            if($data == 'exit')
            {
                // 收到退出指令，关闭子进程
                $worker->exit();
                exit;
            }
            // 模拟任务执行过程
            sleep(mt_rand(1, 2));
            // 执行完成，通知父进程
            $worker->write("" . $worker->pid);
        });
    }
}

new BaseProcess();
// 注册信号，回收退出的子进程
swoole_process::signal(SIGCHLD, function($sig) {
    while($ret =  swoole_process::wait(false)) {
        echo "PID={$ret['pid']}\n";
    }
});
```
这里，父进程初始化的时候创建了3个子进程，然后开始派发任务，每个任务都会找到一个标记为空闲的子进程进行处理。如果发现所有子进程都处于繁忙状态，并且子进程的数量并没有达到最大值，父进程就会创建一个新的子进程接收处理任务。当子进程处理完成后，再通知父进程，将自己的状态设置为空闲，以接收下一次任务。最后，关闭所有子进程，通过signal和wait来回收子进程。

在实际的业务当中，进程池的功能可以更复杂。比如如果不需要动态扩容，可以选择直接创建足够数量的子进程，然后开启消息队列模式，并设置为抢占，派发的任务会在队列中缓存，并自动被处于闲置的子进程获取并处理。再比如如果需要进程池处于闲置时释放不必要的子进程，也可以通过定时器来主动退出一段时间内没有处理任务的子进程。这些例子可以由读者自己尝试完成。

###在Server中使用
在之前的章节中，我们提到过，swoole_server内置了进程管理机制，可以自动管理和重启工作进程。但是我们创建的Process进程并不在Swoole的管理机制内，因此仍然需要手动管理。而在某些特殊的应用场景下，我们又希望能够将创建的Process托管给swoole_server用于执行一些特殊任务。

swoole_server提供了一个addProcess的API用于实现这一功能。该函数的原型如下：
```
/**
 * @param $process  swoole_process      待添加的进程对象
 * @return bool                         添加成功返回true
 */
bool swoole_server::addProcess(swoole_process $process);
```
通过这一函数，我们可以把一个Process对象托管给swoole_server的Manager进程。这样，当Process因意外或错误退出后，Manager进程也会自动创建一个新的Process进程继续执行任务。通过addProcess添加的Process不需要调用start方法或者daemon方法，启动和守护进程化都会有Manager进程完成。
```
$server = new swoole_server('0.0.0.0', 9501);

$process = new swoole_process(function() {
    while(true) {
            // TODO 
            sleep(1);
    }
});

$server->addProcess($process);
$server->start();
```
这里创建的Process进程，必须执行一段会一直运行不退出的逻辑，可以是一个while死循环，也可以是swoole的定时器，但是务必需要保证进程不会因为执行完成退出。因为一旦进程退出，Manager进程会重新创建一个新的进程出来，频繁的关闭创建进程会带来额外的系统开销，因此一定要注意。

文章来自喵星球