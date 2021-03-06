## PHP综合篇2

```
48，并发与并行区别？
并发的关键是你有处理多个任务的能力，不一定要同时。  
并行的关键是你有同时处理多个任务的能力。
并发是轮流处理多个任务，并行是同时处理多个任务

```
```
49,运行一个程序是把程序加载到内存运行，cpu运算:
a.内存存储指令和数据
b.cpu通过指令运行计算生成结果
c.一个程序运行可以生成一个或者多个进程／线程
```

```
50,协程?
a.微线程，程序里面的子程序或者函数，一个线程可以包含多个协程
b.协程是在一个线程执行过程中可以在一个子程序的预定或者随机位置中断，然后转而执行别的子程序，在适当的时候再返回来接着执行。他本身是一种特殊的子程序或者称作函数
c.协程可以解决IO阻塞带来性能问题(比如网络请求, 文件读写等)
d.普通写法, 会遇到 IO阻塞 导致的性能损失
e.单协程: 尽管 IO阻塞 引发了协程调度, 但当前只有一个协程, 调度之后还是执行当前协程
f.多协程: 真正发挥出了协程的优势, 遇到 IO阻塞 时发生调度, IO就绪时恢复运行
```
```
51.TCP／UDP相关

A.TCP 可靠传输，面向连接
    a.三次握手
    客户端发送SYN包到服务器，其中包含客户端的初始序号seq=x，并进入SYN_SENT状态，等待服务器确认。（其中，SYN=1，ACK=0，表示这是一个TCP连接请求数据报文；序号seq=x，表明传输数据时的第一个数据字节的序号是x）
    服务器收到请求后，必须确认客户的数据包。同时自己也发送一个SYN包，即SYN+ACK包，此时服务器进入SYN_RECV状态。（其中确认报文段中，标识位SYN=1，ACK=1，表示这是一个TCP连接响应数据报文，并含服务端的初始序号seq(服务器)=y，以及服务器对客户端初始序号的确认号ack(服务器)=seq(客户端)+1=x+1)
    客户端收到服务器的SYN+ACK包，向服务器发送一个序列号(seq=x+1)，确认号为ack(客户端)=y+1，此包发送完毕，客户端和服务器进入ESTAB_LISHED(TCP连接成功)状态，完成三次握手。
    总结：第三次握手是为了防止：如果客户端迟迟没有收到服务器返回确认报文，这时会放弃连接，重新启动一条连接请求，但问题是：服务器不知道客户端没有收到，所以他会收到两个连接，浪费连接开销。如果每次都是这样，就会浪费多个连接开销。

    b.四次挥手
    首先，客户端发送一个FIN，用来关闭客户端到服务器的数据传送，然后等待服务器的确认。其中终止标志位FIN=1，序列号seq=u。状态 client: fin_wait_1,server:close_wait
    服务器收到这个FIN，它发送一个ACK，确认ack为收到的序号加一 状态 client:fin_wait_2  server:close_wait
    关闭服务器到客户端的连接，发送一个FIN给客户端 状态 client:time_wait       server:last_ack
    客户端收到FIN后，并发回一个ACK报文确认，并将确认序号seq设置为收到序号加一。首先进行关闭的一方将执行主动关闭，而另一方执行被动关闭 状态 client:time_wait     server:last_ack  时间等待:2MSL(最大报文生存时间)  client: closed server: closed
    总结:  双方关闭连接要经过双方都同意。所以，首先是客服端给服务器发送FIN，要求关闭连接，服务器收到后会发送一个ACK进行确认。服务器然后再发送一个FIN，客户端发送ACK确认，并进入TIME_WAIT状态。等待2MSL后自动关闭。

B.UDP 无连接协议，也称透明协议

C.socket
    socket翻译为套接字
    socket是在应用层和传输层之间的一个抽象层，它把TCP/IP层复杂的操作抽象为几个简单的接口供应用层调用以实现进程在网络中通信。socket是一组接口，在设计模式中，socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在socket接口后面，对用户来说，一组简单的接口就是全部，让socket去组织数据，以符合指定的协议
    Socket 其实并不是一个协议。它工作在 OSI 模型会话层（第5层），是为了方便大家直接使用更底层协议（一般是 TCP 或 UDP ）而存在的一个抽象层。
    Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。


D.websocket
    应用层协议
    WebSocket是双向通信协议，模拟Socket协议，可以双向发送或接受信息。HTTP是单向的。
    WebSocket是需要浏览器和服务器握手进行建立连接的。而http是浏览器发起向服务器的连接，服务器预先并不知道这个连接。
    WebSocket在建立握手时，数据是通过HTTP传输的。但是建立之后，在真正传输时候是不需要HTTP协议的。
    
   首先，客户端发起http请求，经过3次握手后，建立起TCP连接；http请求里存放WebSocket支持的版本号等信息，如：Upgrade、 Connection、WebSocket-Version等
   然后，服务器收到客户端的握手请求后，同样采用HTTP协议回馈数据
   最后，客户端收到连接成功的消息后，开始借助于TCP传输信道进行全双工通信。
```
```
52.linux 进程／线程，进程直接相互通信
A.进程与进程之间通过uninx socket通信
B.一个进程包含了一个或多个线程
C.进程:
    c1.一个或多个线程栈
    c2.堆
    c3.数据区
    c4.代码区
```
```
53.微服务之tars:
A.TARS是基于二进制TARS协议的高性能RPC开发框架, 同时配套一体化的运营管理平台, 并通过伸缩调度, 实现运维半托管服务
B.  多语言: TARS支持C++/JAVA/PHP/NodeJS等多种语言;
    敏捷研发: 通过快速服务构建, 自动化代码生成与持续集成工具, 保证研发效率;
    高可用: TARS自带服务发现,容灾容错,智能调度等功能特性
    高效运营: 使用TARS可以实现无损的业务变更, 立体化的监控以及对服务本身可视化的管理
C.不同语言之间, 使用统一的接口定义语言tars来约定协议, 各语言的基本RPC能力均已支持
D.TARS协议和TUP协议
    TARS编码协议是一种数据编解码规则，它将整形、枚举值、字符串、序列、字典、自定义结构体等数据类型按照一定的规则编码到二进制数据流中。对端接收到二进制数据流之后，按照相应的规则反序列化可得到原始数值
    TARS编码协议使用一种叫做TAG的整型值(unsigned char)来标识变量，比如某个变量A的TAG值为100(该值由开发者自定义)，我们将变量值编码的同时，也将该TAG值编码进去。对端需要读取变量A的数值时，就到数据流中寻找TAG值为100的数据段，找到后按规则读出数据部分即是变量A的数值。
    TARS编码协议的定位是一套编码规则。tars协议序列化之后的数据不仅可以进行网络传输，同时还可以存储到数据库中
    TUP组包协议是TARS编码协议的上层封装，定位为通信协议。它使用变量名作为变量的关键字，编码时，客户端将变量名打包到数据流中；解码时，对端根据变量名寻找对应的数据区，然后根据数据类型对该数据区进行反序列化得到原始数值
    TUP组包协议内置一个TARS编码协议的Map类型，该Map的关键字就是变量名，Map的值是将变量的数据值经过TARS编码序列化的二进制数据。
    TUP组包协议封装的数据包可以直接发送给Tars服务端，而服务端可以直接反序列化得到原始值。
    TARS组包协议是对RequestPacket（请求结构体）和ResponsePacket（结果结构体）使用TARS编码协议封装的通信协议。结构体包含比如请求序列号、协议类型、RPC参数序列化之后二进制数据等重要信息
```

```
54.微服务概念（与分布式区别)
A.微服务是架构设计方式，分布式是系统部署方式
B.分布式：一个业务分拆多个子业务，部署在不同的服务器上。集群：同一个业务，部署在多个服务器上
    分布式是以缩短单个任务的执行时间来提升效率的，而集群则是通过提高单位时间内执行的任务数来提升效率
    采用分布式方案，提供 10 台服务器，每台服务器只负责处理一个子任务，不考虑子任务间的依赖关系，执行完这个任务只需一个小时。(这种工作模式的一个典型代表就是 Hadoop 的 Map/Reduce 分布式计算模型）
    而采用集群方案，同样提供 10 台服务器，每台服务器都能独立处理这个任务。假设有 10 个任务同时到达，10 个服务器将同时工作，1 小时后，10 个任务同时完成，这样，整身来看，还是 1 小时内完成一个任务！
    好的设计应该是分布式和集群的结合，先分布式再集群，具体实现就是业务拆分成很多子业务，然后针对每个子业务进行集群部署，这样每个子业务如果出了问题，整个系统完全不会受影响
C.简单来说微服务就是很小的服务，小到一个服务只对应一个单一的功能，只做一件事。这个服务可以单独部署运行，服务之间可以通过RPC来相互交互，每个微服务都是由独立的小团队开发，测试，部署，上线，负责它的整个生命周期。
D.分布式是否属于微服务：
微服务的意思也就是将模块拆分成一个独立的服务单元通过接口来实现数据的交互。
不一定，如果一个很大应用，拆分成三个应用，但还是很庞大，虽然分布式了，但不是微服务。。微服务核心要素是微小。。
F. 微服务的设计是为了不因为某个模块的升级和BUG影响现有的系统业务。
    微服务与分布式的细微差别是，微服务的应用不一定是分散在多个服务器上，他也可以是同一个服务器
    微服务重在解耦合，使每个模块都独立。分布式重在资源共享与加快计算机计算速度。
```
```
55.swoole
A.master进程
    启动主Reactor对象
    多个reactor线程 每个线程一个Reactor对象
    Swoole的Master进程主要负责创建和管理Reactor线程、启动主Reactor对象、创建Manager进程、接收客户端连接请求等工作
    它就是真正处理TCP连接，收发数据的线程。
B.Manger进程
    这个进程没有其他的任务，它仅仅负责创建、回收、管理所有的Worker进程
    子进程结束运行时，manager进程负责回收此子进程，避免成为僵尸进程。并创建新的子进程
    服务器关闭时，manager进程将发送信号给所有子进程，通知子进程关闭服务
    服务器reload时，manager进程会逐个关闭/重启子进程
C.Worker进程
    实际上绝大部分逻辑代码都是在Worker进程中执行的，因此Worker进程所对应的回调函数也是最多的
    接受由Reactor线程投递的请求数据包，并执行PHP回调函数处理数据
    生成响应数据并发给Reactor线程，由Reactor线程发送给TCP客户端
    可以是异步非阻塞模式，也可以是同步阻塞模式
    多进程
D.TaskWorker进程
    任务进程 完全同步阻塞模式
    受由Worker进程通过swoole_server->task/taskwait方法投递的任务
    处理任务，并将结果数据返回（使用swoole_server->finish）给Worker进程
    TaskWorker以多进程的方式运行
```
```
56.面向对象设计模式
A.观察者模式
    观察者观察被观察者，被观察者通知观察者
    当对象间存在一对多关系时，则使用观察者模式（Observer Pattern）。比如，当一个对象被修改时，则会自动通知它的依赖对象。观察者模式属于行为型模式
B.单例模式
    判断系统是否已经有这个单例，如果有则返回，如果没有则创建
    主要解决：一个全局使用的类频繁地创建与销毁
C.工厂模式
    主要解决：主要解决接口选择的问题
    其子类实现工厂接口，返回的也是一个抽象的产品
    1、一个调用者想创建一个对象，只要知道其名称就可以了。 2、扩展性高，如果想增加一个产品，只要扩展一个工厂类就可以。 3、屏蔽产品的具体实现，调用者只关心产品的接口。
    1、您需要一辆汽车，可以直接从工厂里面提货，而不用去管这辆汽车是怎么做出来的，以及这个汽车里面的具体实现。 2、Hibernate 换数据库只需换方言和驱动就可以。
```

```
57,yii2框架的DI依赖注入模式？
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

在yii2.0中，yii\di\Container就是依赖注入容器，这里先简单说一下这个容器的使用。我们可以使用该类的set()方法来注册一个类的依赖，把依赖信息传递给它就可以了，如果希望这个类是单例的，则可以使用setSingleton()方法注册依赖。注册依赖之后，当你需要这个类的对象的时候，使用Yii::createObject()，把类的配置参数传递过去，yii\di\Container即会帮你解决这个类的所有依赖并创建一个对象返回。
```

```
58,hash冲突解决方式？
A,数组是将元素在内存中连续存放。链表中的元素在内存中不是顺序存储的，而是通过存在元素中的指针联系到一起。
B,数组必须事先定义固定的长度，不能适应数据动态地增减的情况。当数据增加时，可能超出原先定义的元素个数；当数据减少时，造成内存浪费。链表动态地进行存储分配，可以适应数据动态地增减的情况
C,(静态)数组从栈中分配空间, 对于程序员方便快速,但是自由度小。链表从堆中分配空间, 自由度大但是申请管理比较麻烦
D,数组和链表在存储数据方面到底孰优孰劣呢？根据数组和链表的特性，分两类情况讨论?
    a,当进行数据查询时，数组可以直接通过下标迅速访问数组中的元素。而链表则需要从第一个元素开始一直找到需要的元素位置，显然，数组的查询效率会比链表的高
    b.当进行增加或删除元素时，在数组中增加一个元素，需要移动大量元 素，在内存中空出一个元素的空间，然后将要增加的元素放在其中。同样，如果想删除一个元素，需要移动大量元素去填掉被移动的元素。而链表只需改动元素中的指针即可实现增加或删除元素。
F,那么，我们开始思考：有什么方式既能够具备数组的快速查询的优点又能融合链表方便快捷的增加删除元素的优势？HASH呼之欲出。
G,所谓的hash，简单的说就是散列，即将输入的数据通过hash函数得到一个key值，输入的数据存储到数组中下标为key值的数组单元中去
H,我们发现，不相同的数据通过hash函数得到相同的key值。这时候，就产生了hash冲突。解决hash冲突的方式有两种。一种是挂链式，也叫拉链法。挂链式的思想在产生冲突的hash地址指向一个链表，将具有相同的key值的数据存放到链表中。另一种是建立一个公共溢出区。将所有产生冲突的数据都存放到公共溢出区，也可以使问题解决
hash表其实是结合了数组和链表的优点，进行的折中方案。平衡了数组和链表的优缺点。hash的具体实现有很多种，但是需要解决冲突的问题。
```



```
59,分布式缓存问题？缓存穿透。。缓存雪崩。。。缓存击穿。。。
A,缓存穿透是指查询一个一定不存在的数据，由于缓存是不命中时需要从数据库查询，查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到数据库去查询，造成缓存穿透。在流量大时，可能DB就挂掉了，要是有人利用不存在的key频繁攻击我们的应用，这就是漏洞。
    a,有很多种方法可以有效地解决缓存穿透问题，最常见的则是采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存在的数据会被这个bitmap拦截掉，从而避免了对底层数据库的查询压力
    b,另外也有一个更为简单粗暴的方法，如果一个查询返回的数据为空（不管是数据不存在，还是系统故障），仍然把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟

B,缓存雪崩是指在设置缓存时采用了相同的过期时间，导致缓存在某一时刻同时失效，导致所有的查询都落在数据库上，造成了缓存雪崩
    a,不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀
    b,在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待

C,缓存击穿是对于一些设置了过期时间的key，如果这些key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题，这个和缓存雪崩的区别在于这里针对某一key缓存，前者则是很多key.
缓存在某个时间点过期的时候，恰好在这个时间点对这个Key有大量的并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮
    a,加锁redis（setnx)、zookeeper 等提供的分布式锁来实现
    
```


