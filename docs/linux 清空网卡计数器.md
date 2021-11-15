## linux 清空网卡计数器
出于要短暂监控某个端口流量的目的，想到要清空某个Linux服务器上网卡的计数器。（木有去搭建监控的条件下）
以前使用cisco设备时候可以 clear counters
linux  这里查看可以使用 ifconfig ethX 看到对应网卡的流量计数器。
清空步骤如下：
 
1.使用 ethtool -i ethx 查看相关网卡的驱动信息。例如查找到驱动为 tg3
2.使用 modeprobe -r tg3;modprobe tg3 可以快速地reload网卡驱动。
3.ifup ethX
4.再次使用ifconfig 查看相应网卡的计数器即可。