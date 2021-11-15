## Swoole理解2


![20180305204653644.jpeg][1]


  [1]: https://www.skylway.com/usr/uploads/2018/08/1289270272.jpeg


swoole使用了reactor设计模式实现高并发：
```
一种基于事件驱动的设计模式，将一个或多个并发服务请求分离（demultiplex）和调度（dispatch）给应用程序。在事件驱动的应用中，同步地、有序地处理同时接收的多个服务请求。 在reactor层面来说依然是同步的，但是对于应用层面他是异步非阻塞的模式。
```

1、reactor
```
swoole使用epoll系列来处理socker网络套接字，他是多路复用IO接口（select/poll）的改进版，epoll有一个专门的消息队列，不需要遍历所有监听的套接字描述符，为高并发的网络链接提供了更好的性能。上图每一个reactor线程都维护了一个reactor模块实例，这个模块就是负责创建epoll实例，监听感兴趣的套接字事件，并设置对应事件处理函数。主线程master获取到一个已连接tcp，并创建一个连接socker套接字后，通过平衡算法将连接socker添加到其中一个reactor线程中监听读写等事件。
```
2、Factory（这并不是一个工厂模式）
```
消息的投递，将reactor线程接收到的数据投递到worker工作进程中处理。worker工作进程将发送给click的数据投递到reactor线程，由reactor线程负责发送给click
```
3、Manager
```
这是一个进程管理模块，worker、task进程异常关闭manger会重新fork一个新的进程。在process模式下负责创建woker进程。
```
4、ReactorThread、ReactorProcess

swoole有三种运行模式：
```
1、base：主进程就是管理进程 ，reactor在worker进程中创建，减少了两次进程间通信。次模式下会使用ReactorProcess

2、process：这个模式就是上图所描述的结构，这个模式运行会调用ReactorThread模块，因为这个模式下reactor是由线程维护的

3、线程模式：用于创建reactor线程，拼接click发送的tcp/dup数据
```
swoole已process模式运行，由主master线程负责accpet从队列中获取已经通过三次握手建立连接的套接字，创建新的连接，将新的链接添加到reactor线程中通过epoll_ctl设置监听事件，当收到客户端信息的时候reactor线程负责接受数据，并在reactor线程中将拼接好的数据通过进程间通信IPC投递到worker进程中，worker负责处理业务逻辑，在woker中可将耗时并且不影响业务逻辑处理的任务放入task进程池中进行处理（这便是swoole提供给php更多灵活性的一点），这样即使是在并发较大的情况下，服务器依然能够接受来自客户端的业务请求。

公司业务中我便是将一些耗时的例如图片处理、日志等任务放入异步队列中。框架是自己实现的，非常简陋，用swoole实现了日志队列。

由于swoole是常驻内存的，这也给php提供了更高的性能，因为不需要每次请求都去加载文件初始化创建资源等，毕竟资源的创建是比较耗时的，例如链接mysql资源，在swoole中mysql链接可以提前创建好，需要用的时候直接从链接池中获取提前创建好的连接对象，而使用apache的方式则是将php作为一个apache的一个模块被加载，每次请求都需要重新加载文件，创建mysql连接。