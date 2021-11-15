## rabbitmq php扩展amqp安装
安装amqp的时候总是提示
```
configure: error: Please reinstall the librabbitmq distribution itself or (re)install librabbitmq development package if it available in your system
```
![请输入图片描述][1]
网上找了一下没有找到解决方法
在看了别人安装amqp的时候发现少安装了一个rabbitmq-c
下面把正常的安装过程分享一下

①安装rabbitmq-c-0.7.1
没有安装就会提示上面的错误
下载地址：https://github.com/alanxz/rabbitmq-c
我选择的是最新版本0.7.1
```
wget https://github.com/alanxz/rabbitmq-c/releases/download/v0.7.1/rabbitmq-c-0.7.1.tar.gz
tar zxf rabbitmq-c-0.7.1.tar.gz

cd rabbitmq-c-0.7.1
./configure --prefix=/usr/local/rabbitmq-c-0.7.1
make && make install
```
成功之后看到如下界面
![请输入图片描述][2]
安装rabbitmq-c成功

②安装amqp
下载地址https://pecl.php.net/package/amqp
我选择的是1.6.1
```
wget http://pecl.php.net/get/amqp-1.9.1.tgz  
tar zxf amqp-1.9.1.tgz
cd amqp-1.9.1

/usr/local/php/bin/phpize

./configure --with-php-config=/usr/local/php/bin/php-config --with-amqp --with-librabbitmq-dir=/usr/local/rabbitmq-c-0.7.1
注意：这里的/usr/local/rabbitmq-c-0.7.1要跟上面rabbitmq-c安装的地址一样

make && make install
```
安装成功之后记录下面的地址，配置添加php模块的时候有用
amqp安装成功

③添加php模块
vi /usr/local/php/etc/php.ini
最后添加一行
extension = /usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/amqp.so
重启php
service php-fpm restart
④检查amqp安装
用phpinfo或者php -m 来检查一下amqp是否安装成功