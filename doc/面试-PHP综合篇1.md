## （PHP综合篇1）

```
PHP 篇
```
```
1,PHP 7 的内存回收原理？ 请详细描述ZendVM的工作原理
  引用计数，refcount = 0 的时候自动回收。
  抽象语法树--》opcodes --》ZendVM编译执行。

  oplines指令 中 opcodes handle编译成机器指令--->执行器运行
```
```
2,PHP 7 的垃圾回收和 PHP 5 有什么区别？

  都属于引用计数的方式来操作。区域在于zval结构变化。

  区别在于php5的引用计数是zval单独存储的，PHP7的引用计数是存在zend_value中的，来自自身数据的存储，放到了数据类型中。
  这种实现方式有以下好处：
  简单数据类型不需要单独分配内存，也不需要计数；
  不会再有两次计数的情况。在对象中，只有对象自身存储的计数是有效的；
  由于现在计数由数值自身存储，所以也就可以和非 zval 结构的数据共享，比如 zval 和 hashtable key 之间；

```
```
3,PHP 7 中对zVal做了哪些修改？
引用计数存储方式不一样 字节数减少了

```
```
4,PHP 7 中哪些变量类型在栈，哪些变量类型在堆？变量在栈会有什么优势？PHP 7是如何让变量新建在栈的？
  局部变量在栈
  对象内容在堆里面，对象名字在栈里面
  变量在栈隔离性，不互相影响，各自运行。栈内存的更新速度很快，因为局部变量的生命周期都很短，有自己的作用域，一旦离开作用域，变量就会被释放。

  新建一个局部变量默认就在栈里面。

  栈内存是可以直接存取的，而堆内存是 不可以直接存取的内存。对于我们的对象来数就是一种大的数据类型而且是占用空间不定长的类型，所以说对象是放在堆里面的，但对象名称是放在栈里面的，这样通过对象名称就可 以使用对象了
```
```
5,详细描述PHP中HashMap的结构是如何实现的？下面代码中，在PHP 7下， $a 和 $b、$c、$d 分别指向什么zVal结构？$d 被修改的时候，PHP 7 / PHP 5 的内部分别会有哪些操作？
	$a = 'string';
	$b = &$a;
	$c = &$b;
	$d = $b;
	$d = 'to';

  注意这里$a的类型在&操作后已经转为引用
```
```
6,JIT 是做了哪些优化，从而对PHP的速度有不少提升？

  对一些频繁使用的代码块运行时编译成机器指令
```
```
7,strtr 和 str_replace 有什么区别，两者分别用在什么场景下？strtr的程序是如何实现的？

  a,strtr是等长度转换,不能以少换多或者以多换少
  b,strtr不能转换为空
  c,strtr效率相对快一些 据说，strtr 比 str_replace 快四倍
  d,strtr是转换，str_replace是替换
  

  echo strtr("Hilla Ward","ia","eo"); 结果是Hello Word
  echo str_replace("world","baidu","Hello world!"); 结果是 Hello baidu!
```
```
8,字符串在手册中介绍，「PHP的字符串是二进制安全的」，这句话怎么理解，为什么是二进制安全？
  例如strlen，在输入数据里有\0的时候，并不会在此停止。所以可以说是二进制安全的。
  PHP代码都会被zend引擎编译成opcode，最终作为C语言去执行。 而对于c语言‘\0’是字符串的结束符，它读到’\0’就会默认字符读取已经结束，从而抛掉后面的字符串
```
```
9,字符串连接符.，在PHP内核中有哪些操作？多次.连接，是否会造成内存碎片过多？
```
```
10,PHP中创建多进程有哪些方式？互斥信号该如何实现？
  pcntl pcntl_fork swoole:process。 lock 加锁：flock($fp, LOCK_EX) 释放锁：flock($fp, LOCK_UN); 
  利用文件锁 redis锁setnx
```
```
11,swoole服务端启动后有哪些进程，这些进程分别是完成什么工作？
master manager worker task
```
```
12,线上环境中，PHP进程偶尔会卡死（或者运行卡顿），请问如何检测本质问题？
gdb php server.php gdb -p pid gdb php core
strace -p pid
打印日志display_errors=on;
```
```
13,实现管道的makeFn函数？
function pipe($input, $list) {
    $fn = makeFn($list); 
    return $fn($input);
}
$r = pipe(0, [$a, $b, $c]);
echo $r;

//$a, $b, $c 类似于
$a = function($input, $next) {
    $input++;
    $output = $next($input);
    return $output;
};

function makeFn($list){
    //请实现

}
```
```

14，用PHP实现一个定时任务器，类似crontab，需要做到前一个任务不论运行时长、运行失败，都不能影响下一个任务的准点执行？
swoole_timer
pcntl_alarm
```
```
15，PHP中密码加密，使用什么方式加密？这种加密的优点是什么？
md5 crypt(des) sha1 urlencode base_encode

```
```
16，实现如下函数(PHP 7)

echo a(1, 3); //4
echo a(3)(5); //8
echo a(1, 2)(3, 4, 5)(6); //21

```
```
17，如何读取某函数的参数列表，以及参数的默认值。

func_get_args
```
```
18，如何模拟Java的注解方法，比如识别如下代码中的路由

class Controller {
     /**
      * @Route("/", name="index")
      * @CheckRequest
      */
     public function index(Request $request){
         return 'result';
     }
}
```
```

19，描述下IoC （DI）的实现原理？依赖注入？
A,为什么需要依赖注入?
    首先我们先不管什么是依赖注入，先来分析一下没有使用依赖注入会有什么样的结果。假设我们有一个gmail邮件服务类GMail，然后有另一个类User，User类需要使用发邮件的功能，于是我们在User类中定义一个成员变量$mailServer，并且在声明这个变量的时候就给它赋值一个GMail类对象，或者在User构造函数中进行GMail类实例化与赋值。这样写程序会有什么问题呢？试想一下，每次当我们需要把User使用的邮件服务改为其他类型邮件服务的时候，我们需要频繁修改User类的成员变量$mailServer，这样是不好的。问题的根源就在于，我们不该把User类的成员变量$mailServer的实例化写死在User类内部，而应该在调用User类的时候可以动态决定赋值给$mailServer的对象类型，依赖注入就是来解决这个问题的
B,依赖注入是什么?
    所谓依赖注入，实质上就是当某个类对象需要使用另一个类实例的时候，不在类内部实例化另一个类，而将实例化的过程放在类外面实现，实例化完成后再赋值给类对象的某个属性。这样的话该类不需要知道赋值给它的属性的对象具体属于哪个类的，当需要改变这个属性的类型的时候，无需对这个类的代码进行任何改动，只需要在使用该类的地方修改实例化的代码即可。
C,依赖注入的方式有两种：
    a.构造函数注入，将另一个类的对象作为参数传递给当前类的构造函数，在构造函数中给当前类属性赋值；
    b.属性注入，可以将该类某个属性设置为public属性，也可以编写这个属性的setter方法，这样就可以在类外面给这个属性赋值了。
D,依赖注入容器
    仔细思考一下，我们会发现，虽然依赖注入解决了可能需要频繁修改类内部代码的问题，但是却带来了另一个问题。每次我们需要用到某个类对象的时候，我们都需要把这个类依赖的类都实例化，所以我们需要重复写这些实例化的代码，而且当依赖的类又依赖于其他类的时候，我们还要找出所有依赖类依赖的其他类然后实例化，可想而知，这是一个繁琐低效而又麻烦且容易出错的过程。这个时候依赖注入容器应运而生，它就是来解决这个问题的。
    依赖注入容器可以帮我们实例化和配置对象及其所有依赖对象，它会递归分析类的依赖关系并实例化所有依赖，而不需要我们去为这个事情费神。
```
```
20，描述XSS注入原理，以及如何防止？
A,XSS又叫CSS (Cross Site Script) ，跨站脚本攻击。它指的是恶意攻击者往Web页面里插入恶意脚本代码，而程序对于用户输入内容未过滤，当用户浏览该页之时，嵌入其中Web里面的脚本代码会被执行，从而达到恶意攻击用户的特殊目的。
B,假如我在评论里输入了一段可执行的script，让他弹出 123，当我把评论提交，如果没有网站没有做处理的话，那么我的脚本就会当成评论展示给其他人看，但是它在页面执行了，然后大伙一到我的评论页面就会弹出 123
C,窃取cookie、放蠕虫、网站钓鱼
D，防止：
    过滤输入 
    addslashes，不要相信输入，做好字段过滤和校验
    mysql_real_escape_string()
```
```
21，描述Csrf注入原理，以及如何防止？
A,CSRF概念：CSRF跨站点请求伪造(Cross—Site Request Forgery)，跟XSS攻击一样，存在巨大的危害性，你可以这样来理解：
B,你这可以这么理解CSRF攻击：攻击者盗用了你的身份，以你的名义发送恶意请求。CSRF能够做的事情包括：以你名义发送邮件，发消息，盗取你的账号，甚至于购买商品，虚拟货币转账......造成的问题包括：个人隐私泄露以及财产安全
防御CSRF攻击：
    a,验证 HTTP Referer 字段
    b,在请求地址中添加 token 并验证
    c,在 HTTP 头中自定义属性并验证
```
```
22，ES 6中的 Promise 对象是做什么的？


23，解释ES 6中async、await的使用场景？

24，ES 6中 遍历器Iterator 怎么写，其作用是什么？ 回调地狱(callback hell) 如何使用 遍历器Iterator 实现，提示：Thunk

25，写出下面代码执行后输出的内容

var p1 = new Promise(resolve => {
    console.log(1);
    resolve(2);
})
let p2 = new Promise(resolve => {
    console.log(3);
    resolve(p1);
});
p1.then(re => {
    console.log(re);
});
p2.then(re => {
    console.log(re);
});



26，vue 和 angularJS 中检测脏数据的原理有什么区别？

27，vue中，vuex的主要作用是什么？

28，vue中 data 和computed 有什么区别？
{
    computed: {
        now() {
            return new Date();
        }
    }
}
上面的now变量，是否能够在每次调用时得到当前时间？




29，vuex中mutations 和actions 有什么区别？


30，vuex中如何在外部（可以理解为任意一段<script>中）设置变量的值，以及如何调用mutations

```
```
通讯协议篇
```
```
31，详细描述 HTTPS（SSL）工作原理？
    HTTPS（Hyper Text Transfer Protocol Secure），即超文本传输安全协议，也称为http over tls等，是一种网络安全传输协议，其也相当于工作在七层的http，只不过是在会话层和表示层利用ssl/tls来加密了数据包，访问时以https://开头，默认443端口，同时需要证书，学习https的原理其实就是在学习ssl/tls的原理
    SSL证书机构
```
```
32，服务器使用PHP时，客户端的IP能伪造吗？如果能，列出伪造方法；如果不能，说明原因？
    
    CURD 代理访问
```
```
33，描述域名劫持的各种方法，为什么HTTPS不能被劫持？

    域名劫持是互联网攻击的一种方式，通过攻击域名解析服务器（DNS），或伪造域名解析服务器（DNS）的方法，把目标网站域名解析到错误的地址从而实现用户无法访问目标网站的目的

    请求dns服务器获取域名对应的ip,然后在通过ssl和服务器建立安全通道吗？如果发生域名劫持，dns服务器返回错误的ip地址，由于需要证书认证问题，会报网站无法访问错误
```
```
34，描述HTTP协议是什么，以及HTTP 2 和 HTTP 1.1 有什么区别？
```
```
35，详细描述IP协议、TCP协议，以及UDP协议与它们的区别。
```
```
36，TCP协议中，最大传输单元MTU一般最大是多少，在TCP协议中，如果一个数据被分割成多个包，这些包结构中什么字段会被标记相同。UDP分包和TCP分包会有哪些区别？
```
```
37，HTTP协议中 Transfer-Encoding: Chunked 适用于哪些应用场景，这个与使用Content-Length: xxx在收到的报文包上有哪些区别？

```

```
分布式篇
```

```
38，描述epoll和poll、select的区别，为什么epoll会具备性能优势？
  a,这三种都是是Linux API提供的I/O复用方式（在Linux Socket服务器短编程时，为了处理大量客户的连接请求，需要使用非阻塞I/O和复用）
  b,select函数监视的文件方式是参数-值”传递的，调用后select函数会阻塞，直到有描述符就绪,当select函数返回后，可以 通过遍历fdset，来找到就绪的描述符(需要遍历，还有阻塞)。数量存在最大限制，1024。
  c,poll不再使用select“参数-值”传递的方式，采用pollfd结构，改用监视的event和发生的event的方式，pollfd并没有最大数量限制（但是数量过大后性能也是会下降）。 和select函数一样，poll返回后，需要轮询pollfd来获取就绪的描述符。
  d,epoll的优点:IO的效率不会随着监视fd的数量的增长而下降。epoll不同于select和poll轮询的方式，而是通过每个fd定义的回调函数来实现的。只有就绪的fd才会执行回调函数,监视的描述符数量不受限制。
```
```
39，描述下惊群的原因？有什么有效的方法可以避免惊群？
    在多进程/多线程等待同一资源时，也会出现惊群。即当某一资源可用时，多个进程/线程会惊醒，竞争资源。这就是操作系统中的惊群。
    加锁，保证同一时间只有一个线程处理一个连接
```
```
40，什么是Hash一致性，这个方法主要运用在什么场景？如何实现增加新节点之后，整体做最小的数据移动 
    redis集群  节点:ip % 2^32 和 值存储节点:key % 2^32 环中取节点,然后顺时针取最近的节点。自己代码实现。
    新增或下线服务器，也不会影响全部，只要根据hash顺时针定位就可以了。
    自己做虚拟节点整体做最小的数据移动
```
```
41，有哪些分布式锁？
    redis setnx
    zookeeper
```
```
42，ZooKeeper 能解决哪些问题？具体说明。
   a,统一命名服务
   b,配置管理
   c,集群管理 集群worker管理 zookeeper监控集群
   d,分布式锁 主要得益于ZooKeeper为我们保证了数据的强一致性，zookeeper的znode节点创建的唯一性和递增性能保证所有来抢锁的worker的原子性
```
```
综合篇
```
```
43，描述OAuth2的工作原理？
    客户端请求登录--服务端验证登录成功回掉客户端地址下发token给客户端使用，有时效性。
```
```
44，Swoole 中协程实现原理，以及为什么会提升效率？
    Reactor协程负责事件监听，在IO事件完成后唤醒其他工作协程
    可以多协程运行程序，在子程序代码里面，运行后挂起转到其它子程序继续运行新子程序逻辑。类似并行处理子程序的逻辑。

    4.0协程实现中，主协程即为Reactor协程，负责整个EventLoop的运行。主协程实现事件监听，在IO事件完成后唤醒其他工作协程
    协程挂起：在工作协程中执行一些IO操作时，底层会将IO事件注册到EventLoop，并让出执行权。
    协程恢复：当主协程的Reactor接收到新的IO事件，底层会挂起主协程，并恢复IO事件对应的工作协程。该工作协程挂起或退出时，会再次回到主协程。
```
```
45，列出几个中文分词工具？
    HTTPCWS – 基于HTTP协议的开源中文分词系统
    SCWS – 简易中文分词系统
    PhpanAlysis - PHP无组件分词系统
    ICTCLAS – 全球最受欢迎的汉语分词系统
```
```
46，git 放弃未提交的文件有哪些方法？git删除远程分支、Tag有什么方法？git覆盖远程仓库有什么办法？
  git reset 
  git checkout .
  git clean -fdx
  git branch -D
  git tag -d

  删除远程分支
  git branch -r -d origin/branch-name
  git push origin :branch-name

  删除远程Tag
  显示本地 tag
  git tag 
  Remote_Systems_Operation
  删除本地tag
  git tag -d Remote_Systems_Operation 
  用push, 删除远程tag
  git push origin :refs/tags/Remote_Systems_Operation


  git覆盖远程仓库
  git push -u origin develop
```
```
47，CentOS 下安装php扩展有哪些方法？
pecl install xxx 
phpize make && make install
```

