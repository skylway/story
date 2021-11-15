### nginx+php-fpm故障排查


看到这篇文章写的不错，转载分享给大家，故事里面的小明是一个很认真的人，哈哈！！


小明初到一家公司做运维的工作，刚来的第一天就开始部署LNMP(Linux+Nginx+MySQL+PHP)环境，结果出现了问题。 
他来向我请教，具体问题现象、原因和解决思路如下:
问题一:
```
nginx进程CPU和内存不均衡，某个进程占用资源特别高，如何解决？

```
回答：我让小明绑定下CPU的亲缘性(设置nginx配置worker_cpu_affinity项为auto，auto这个特殊值(1.9.10版本)允许自动绑定工作进程到可用的CPU上。)，绑定后CPU和内存使用就均衡了。
问题二：
```
小明又问close系统调用消耗很高怎么解决？
```

回答：且听我娓娓道来，继续看下文。

$ strace -cp $(pgrep -n nginx)

![请输入图片描述][1]


![请输入图片描述][2]

系统32c的，top查看负载去到75.14，

查看过nginx和php-fpm的

错误日志也没有什么发现。

strace 跟踪close的系统调用
都是以下的信息：

$ strace -T -ttp $(pgrep -n nginx) 2&>1 |grep -B 10 close > ./close.log

![请输入图片描述][3]

$ lsof | more

![请输入图片描述][4]

遂怀疑是连接创建关闭消耗了太多的资源，便让小明加了台机器专门跑nginx，
前端挂了个nginx用长连接跟后端的nginx连接，接了个nginx之后负载果然就下来了
![请输入图片描述][5]

前端未挂nginx压测ab压测结果:

![请输入图片描述][6]

前端挂了nginx压测ab压测结果:

![请输入图片描述][7]
tps基本没变第一个Time per，requset快了87.52%。
接着继续排查tps上不去的原因，继续strace后端的nginx。

$ strace -cp $(pgrep -n nginx)

![请输入图片描述][8]

发现现在是wrtiev占用高了，strace 跟踪close的系统调用，
发现很多以下的输出：
connect(26, {sa_family=AF_INET, 
sin_port=htons(9000), 
sin_addr=inet_addr("127.0.0.1")},
 16) = -1 EINPROGRESS
 (Operation now in progress)

问题应该不是在nginx上，
应该是在php-fpm上了。

继续

$ strace -cp $(pgrep -n php-fpm)

显示下图所示：
![请输入图片描述][9]


access cpu时间消耗最多那就先
排查access
系统调用：
$ strace -T -ttp 
$(pgrep -n php-fpm) 2&>1 |
grep -B 10 access >
 ./access.log

![请输入图片描述][10]


php-fpm进程频繁的去读取文件，整个操
作下来花费4ms的时间。

然后排查recvfrom：
$ strace -T -ttp $(pgrep -n 
php-fpm) 2&>1 |
grep -B 10 recvfrom > 
./recvfrom.log


![请输入图片描述][11]
频繁的去访问10.0.0.156的6379,端口，明显就是访问redis读取数据的过程，
整个过程花费12ms。
让小明把上面两个strace信息发给开发，第一个得到回复是老版本的流程，
新版本改了，但还是有些判断没有处理。
第二个问题让开发使用redis连接池，无需频繁创建连接读取数据，
频繁创建连接开销很大的。

当遇上性能问题时，排查日志无法解决时，使用strace工具来查看一下系统调用，看时间到底消耗在哪里了，可以轻松的找到问题所在。


  [1]: https://mmbiz.qpic.cn/mmbiz_png/emWJs4asicsBXYcU2GicD6QXeljYyXiaUcEic8rdYTYycGmvOg7PJqhiaYJAnrJ1nJjfLaL0sGdZKnsTm6duxPefOuQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1
  [2]: https://mmbiz.qpic.cn/mmbiz_jpg/emWJs4asicsBXYcU2GicD6QXeljYyXiaUcElfn5aVicb7v6iceH1V9wvupnTdRUkUCq2b7XjnRexgP4vzQlWqlhK2KQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1
  [3]: https://mmbiz.qpic.cn/mmbiz_png/emWJs4asicsBXYcU2GicD6QXeljYyXiaUcEiahxUQiaiaW9ibE1vBV5VJMDYaksJQibJ2esv8Q4er7kU7XKicqvw4Vl5oKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1
  [4]: https://mmbiz.qpic.cn/mmbiz_png/emWJs4asicsBXYcU2GicD6QXeljYyXiaUcEldicBU0gQlOFF9Cgj7xm8bs0KkYnVbTAkbWSDRoZyFDlePqj5cKGoibQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1
  [5]: https://mmbiz.qpic.cn/mmbiz_png/emWJs4asicsBXYcU2GicD6QXeljYyXiaUcEmqTh40iaf8T1pyHvnQ2DNCXhhdOTbp4Pqhs5GCriaLhknmup4QutDViaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1
  [6]: https://mmbiz.qpic.cn/mmbiz_png/emWJs4asicsBXYcU2GicD6QXeljYyXiaUcEDuibicyCKxTyftexmUZo6wN9auM3aXRIvXnwibh6Z2ldrDEvbnLiaa57IA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1
  [7]: https://mmbiz.qpic.cn/mmbiz_png/emWJs4asicsBXYcU2GicD6QXeljYyXiaUcEy7icibBGS0JHFUgDmG7A0BAMq2OtQoPAS5lQN4vCInbPKlJRQu5r7Z8A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1
  [8]: https://mmbiz.qpic.cn/mmbiz_png/emWJs4asicsBXYcU2GicD6QXeljYyXiaUcEaicrXZuJrC3FxZjhZI43OcxdUEvJ61jwKytO4nvkdo3fDBxB3N7kGTQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1
  [9]: https://mmbiz.qpic.cn/mmbiz_jpg/emWJs4asicsBXYcU2GicD6QXeljYyXiaUcE3uibq9d0FductvwMtalc8rgYib1VvwmibezOwN8Hcr2vVEyNnuV4TvpGg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1
  [10]: https://mmbiz.qpic.cn/mmbiz_png/emWJs4asicsBXYcU2GicD6QXeljYyXiaUcEgDntwI4gKiaXcgF2kp1Uibf6pWBfZCicaYCVVjOI8FiasZyWiaGQ7LxJVUg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1
  [11]: https://mmbiz.qpic.cn/mmbiz_png/emWJs4asicsBXYcU2GicD6QXeljYyXiaUcEukhhjEvkJakZq5N7qRFEmpxWDmCDK4hI3qJT3QDuaJRBlKiau4ENShA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1