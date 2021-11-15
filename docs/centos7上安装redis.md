## centos7上安装redis

centos7上安装redis
关闭防火墙：
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
firewall-cmd --state #查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
配置编译环境：
sudo yum install gcc-c++
下载源码：
wget http://download.redis.io/releases/redis-3.2.8.tar.gz
解压源码：
tar -zxvf redis-3.2.8.tar.gz
进入到解压目录：
cd redis-3.2.8
执行make编译Redis：
make MALLOC=libc
注意：make命令执行完成编译后，会在src目录下生成6个可执行文件，分别是redis-server、redis-cli、redis-benchmark、redis-check-aof、redis-check-rdb、redis-sentinel。
安装Redis：
make install 
配置Redis能随系统启动:
./utils/install_server.sh
显示结果信息如下：
Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379] 
Selecting default: 6379
Please select the redis config file name [/etc/redis/6379.conf] 
Selected default - /etc/redis/6379.conf
Please select the redis log file name [/var/log/redis_6379.log] 
Selected default - /var/log/redis_6379.log
Please select the data directory for this instance [/var/lib/redis/6379] 
Selected default - /var/lib/redis/6379
Please select the redis executable path [/usr/local/bin/redis-server] 
Selected config:
Port           : 6379
Config file    : /etc/redis/6379.conf
Log file       : /var/log/redis_6379.log
Data dir       : /var/lib/redis/6379
Executable     : /usr/local/bin/redis-server
Cli Executable : /usr/local/bin/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort.
Copied /tmp/6379.conf => /etc/init.d/redis_6379
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!

Redis服务查看、开启、关闭:
a.通过ps -ef|grep redis命令查看Redis进程
b.开启Redis服务操作通过/etc/init.d/redis_6379 start命令，也可通过（service redis_6379 start）
c.关闭Redis服务操作通过/etc/init.d/redis_6379 stop命令，也可通过（service redis_6379 stop）

redis.conf 的配置信息
1、daemonize 如果需要在后台运行，把该项改为yes
2、pidfile 配置多个pid的地址 默认在/var/run/redis.pid
3、bind 绑定ip，设置后只接受来自该ip的请求
4、port 监听端口，默认是6379
5、loglevel 分为4个等级：debug verbose notice warning
6、logfile 用于配置log文件地址
7、databases 设置数据库个数，默认使用的数据库为0
8、save 设置redis进行数据库镜像的频率。
9、rdbcompression 在进行镜像备份时，是否进行压缩
10、dbfilename 镜像备份文件的文件名
11、Dir 数据库镜像备份的文件放置路径
12、Slaveof 设置数据库为其他数据库的从数据库
13、Masterauth 主数据库连接需要的密码验证
14、Requriepass 设置 登陆时需要使用密码
15、Maxclients 限制同时使用的客户数量
16、Maxmemory 设置redis能够使用的最大内存
17、Appendonly 开启append only模式
18、Appendfsync 设置对appendonly.aof文件同步的频率（对数据进行备份的第二种方式）
19、vm-enabled 是否开启虚拟内存支持 （vm开头的参数都是配置虚拟内存的）
20、vm-swap-file 设置虚拟内存的交换文件路径
21、vm-max-memory 设置redis使用的最大物理内存大小
22、vm-page-size 设置虚拟内存的页大小
23、vm-pages 设置交换文件的总的page数量
24、vm-max-threads 设置VM IO同时使用的线程数量
25、Glueoutputbuf 把小的输出缓存存放在一起
26、hash-max-zipmap-entries 设置hash的临界值
27、Activerehashing 重新hash