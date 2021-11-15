在MySQL或MariaDB中，任意时间对数据库所做的修改，都会被记录到日志文件中。例如，当你添加了一个新的表，或者更新了一条数据，这些事件都会被存储到二进制日志文件中。二进制日志文件在MySQL主从复合中是非常有用的，主服务器会发送其数据到远程服务器中。
当你需要恢复MySQL时，也会需要使用到二进制日志文件。

mysqlbinlog 命令，以用户可视的方式展示出二进制日志中的内容。同时，也可以将其中的内容读取出来，供其他MySQL实用程序使用。

在此示例中，我们将会涉及以下内容：

获取当前二进制日志列表
mysqlbinlog默认行为
获取特定数据库条目
禁止恢复过程产生日志
在输出中控制base-64 BINLOG
mysqlbinlog输出调试信息
跳过前N个条目
保存输出到文件
从一个特定位置提取条目
将条目截止到一个特定的位置
刷新日志以清除Binlog输出
在输出中只显示语句
查看特定开始时间的条目
查看特定结束时间的条目
从远程服务器获取二进制日志
1 获取当前二进制日志列表
在mysql中执行以下命令，即可查看二进制日志文件的列表。

```
mysql> SHOW BINARY LOGS;
+----------------------+----------+
| Log_name              | File_size | 
+--------------------------+------------+
| mysqld-bin.000001 |     15740 |
| mysqld-bin.000002 |       3319 | 
.. 
..
```

如果熊没有开启此功能，则会显示：

```
mysql> SHOW BINARY LOGS;
ERROR 1381 (HY000): You are not using binary logging
```
二进制日志文件默认会存放在 /var/lib/mysql 目录下

```
$ ls -l /var/lib/mysql/
-rw-rw----. 1 mysql mysql 15740 Aug 28 14:57 mysqld-bin.000001
-rw-rw----. 1 mysql mysql  3319 Aug 28 14:57 mysqld-bin.000002
.. 
..
```
2 mysqlbinlog 默认行为
下面将以一种用户友好的格式显示指定的二进制日志文件（例如:mysqld.000001）的内容。
```
$ mysqlbinlog mysqld-bin.000001
```
mysqlbinlog默认会显示为以下内容：
```
/*!40019 SET @@session.max_insert_delayed_threads=0*/;/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/; DELIMITER /*!*/;# at 4#170726 14:57:37 server id 1  end_log_pos 106   Start: binlog v 4, server v 5.1.73-log created 170726 14:57:37 at startup# Warning: this binlog is either in use or was not closed pro<a href="http://www.ttlsa.com/perl/" title="perl"target="_blank">perl</a>y.
ROLLBACK/*!*/; 
BINLOG ' IeZ4WQ8BAAAAZgAAAGoAAAABAAQANS4xLjczLWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA AAAAAAAAAAAAAAAAAAAh5nhZEzgNAAgAEgAEBAQEEgAAUwAEGggAAAAICAgC '/*!*/;
# at 106
#170726 14:59:31 server id 1  end_log_pos 182   Query   thread_id=2     exec_time=0     error_code=0
SET TIMESTAMP=1501095571/*!*/;
SET @@session.pseudo_thread_id=2/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=1, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=0/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C latin1 *//*!*/;
SET @@session.character_set_client=8,@@session.collation_connection=8,@@session.collation_server=8/*!*/;
 ..
 ..
 ..
# at 14191
#170726 15:20:38 server id 1  end_log_pos 14311         Query   thread_id=4     exec_time=0     error_code=0SET TIMESTAMP=1501096838/*!*/;
insert into salary(name,dept) values('Ritu', 'Accounting')/*!*/; 
DELIMITER ;
# End of log file
ROLLBACK /* added by mysqlbinlog */;
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*
```

上面的命令将会显示出，在该系统上数据库发生的所有改变事件。

3 获取特定数据库条目

默认情况下，mysqlbinlog会显示所有的内容，太过于杂乱。使用 -d 选项，可以指定一个数据库名称，将只显示在该数据库上所发生的事件。
```
$ mysqlbinlog -d crm mysqld-bin.000001 > crm-events.txt
```
也可以使用 --database 命令，效果相同。
```
$ mysqlbinlog -database crm mysqld-bin.000001 > crm-events.txt
```
4 禁止恢复过程产生日志
在使用二进制日志文件进行数据库恢复时，该过程中也会产生日志文件，就会进入一个循环状态，继续恢复该过程中的数据。因此，当使用mysqlbinlog命令时，要禁用二进制日志，请使用下面所示的-D选项：

```
$ mysqlbinlog -D mysqld-bin.000001
```
也可以使用 --disable-log-bin 命令，效果相同。
```
$ mysqlbinlog --disable-log-bin mysqld-bin.000001
```

备注：在输出中，当指定-D选项时，将看到输出中的第二行。也就是SQL_LOG_BIN=0
```
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!32316 SET @OLD_SQL_LOG_BIN=@@SQL_LOG_BIN, SQL_LOG_BIN=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
```
当使用-to-last-log选项时，这个选项也会有所帮助。另外，请记住，该命令需要root权限来执行。

5 在输出中控制base-64 BINLOG

使用base64-output选项，可以控制输出语句何时是输出base64编码的BINLOG语句。以下是base64输出设置的可能值：
never
always
decode-rows
auto（默认）
never：当指定如下所示的“never”时，它将在输出中显示base64编码的BINLOG语句。
```
$ mysqlbinlog --base64-output=never mysqld-bin.000001
```
将不会有任何与下面类似的行，它具有base64编码的BINLOG。
```
BINLOG ' IeZ4WQ8BAAAAZgAAABAAQANS4xLjczLWxvZwAAAAAAAAAAAAAAAAAAA AAAAAAAAAAAAAAh5nhZEzgNAAgAEgAEBAQEEgAAUwAEGggAAAAICAgC
```
always：当指定“always”选项时，只要有可能，它将只显示BINLOG项。因此，只有在专门调试一些问题时才使用它。
```
$ mysqlbinlog --base64-output=always mysqld-bin.000001
```
下面是“always”的输出，它只显示了BINLOG项。
```
BINLOG ' IeZ4WQ8BAAAAZgAAAGoAAAABAAQANS4xLjczLWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA AAAAAAAAAAAAAAAAAAAh5nhZEzgNAAgAEgAEBAQEEgAAUwAEGggAAAAICAgC '/*!*/;
# at 106
#170726 14:59:31 server id 1  end_log_pos 182
BINLOG ' k+Z4WQIBAAAATAAAALYAAAAIAAIAAAAAAAAADAAAGgAAAEAAAAEAAAAAAAAAAAYDc3RkBAgACAAI AHRoZWdlZWtzdHVmZgBCRUdJTg== '/*!*/;
# at 182
#170726 14:59:30 server id 1  end_log_pos 291
BINLOG ' kuZ4WQIBAAAAbQAAACMBAAAAAAIAAAAAAAAADAAAGgAAAEAAAAEAAAAAAAAAAAYDc3RkBAgACAAI AHRoZWdlZWtzdHVmZgBJTlNFUlQgSU5UTyB0IFZBTFVFUygxLCAnYXBwbGUnLCBOVUxMKQ== '/*!*/;
# at 291
#170726 14:59:30 server id 1  end_log_pos 422
BINLOG ' kuZ4WQIBAAAAgwAAAKYBAAAAAAIAAAAAAAAADAAAGgAAAEAAAAEAAAAAAAAAAAYDc3RkBAgACAAI AHRoZWdlZWtzdHVmZgBVUERBVEUgdCBTRVQgbmFtZSA9ICdwZWFyJywgZGF0ZSA9ICcyMDA5LTAx LTAxJyBXSEVSRSBpZCA9IDE=
```
decode-rows：这个选项将把基于行的事件解码成一个SQL语句，特别是当指定-verbose选项时，如下所示。

```
$ mysqlbinlog --base64-output=decode-rows --verbose mysqld-bin.000001
```

auto：这是默认选项。当没有指定任何base64解码选项时，它将使用auto。在这种情况下，mysqlbinlog将仅为某些事件类型打印BINLOG项，例如基于行的事件和格式描述事件。

```
$ mysqlbinlog --base64-output=auto mysqld-bin.000001
$ mysqlbinlog mysqld-bin.000001
```

6 mysqlbinlog输出调试信息

下面的调试选项，在完成处理给定的二进制日志文件之后，将检查文件打开和内存使用。
```
$ mysqlbinlog --debug-check mysqld-bin.000001
```
如下所示，在完成处理给定的二进制日志文件之后，下面的调试信息选项将显示额外的调试信息。
```
$ mysqlbinlog --debug-info mysqld-bin.000001 > /tmp/m.di
User time 0.00, System time 0.00
Maximum resident set size 2848, Integral resident set size 0
Non-physical pagefaults 863, Physical pagefaults 0, Swaps 0
Blocks in 0 out 48, Messages in 0 out 0, Signals 0
Voluntary context switches 1, Involuntary context switches 2
```

7 跳过前N个条目

除了读取整个mysql二进制日志文件外，也可以通过指定偏移量来读取它的特定部分。可以使用 -o 选项。o代表偏移。
下面将跳过指定的mysql bin日志中的前10个条目
```
$ mysqlbinlog -o 10 mysqld-bin.000001
```

为了确保它正常工作，给偏移量提供一个巨大的数字，将看不到任何条目。下面的内容将从日志中跳过10,000个条目(事件)。

```
$ mysqlbinlog -o 10000 mysqld-bin.000001
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/; .. ..
# End of log file
ROLLBACK /* added by mysqlbinlog */;
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
```

在本例中，由于这个特定的日志文件没有10,000个条目，所以在输出中没有显示任何数据库事件。


8 保存输出到文件

也可以使用简单的Linux重定向命令，将输出存储到一个文件中，如下所示。
```
$ mysqlbinlog mysqld-bin.000001 > output.log
```
或者也可以使用 -r （结果文件）选项，如下所示，将输出存储到一个文件中。
```
$ mysqlbinlog -r output.log mysqld-bin.000001
```
备注：还可以使用 -server-id 指定mysql服务器，确保是由给定服务器id的mysql服务器所生成的日志。
```
$ mysqlbinlog --server-id=1 -r output.log mysqld-bin.000001
```


9 从一个特定位置提取条目

通常在mysql二进制日志文件中，你将看到如下所示的位置号。下面是mysqlbinlog的部分输出，你可以看到“15028”是一个位置编号。


```
#170726 15:38:14 server id 1  end_log_pos 15028         Query   thread_id=5     exec_time=0     error_code=0
SET TIMESTAMP=1501097894/*!*/;
insert into salary values(400,'Nisha','Marketing',9500)
/*!*/;
# at 15028
#170726 15:38:14 server id 1  end_log_pos 15146         Query   thread_id=5     exec_time=0     error_code=0
SET TIMESTAMP=1501097894/*!*/;
insert into salary values(500,'Randy','Technology',6000)
```

下面的命令将从位置编号为15028的二进制日志条目处开始读取。

```
$ mysqlbinlog -j 15028 mysqld-bin.000001 > from-15028.out
```
当在命令行中指定多个二进制日志文件时，开始位置选项将仅应用于给定列表中的第一个二进制日志文件。还可以使用 -H 选项来获得给定的二进制日志文件的十六进制转储，如下所示。

```
$ mysqlbinlog -H mysqld-bin.000001 > binlog-hex-dump.out
```

10 将条目截止到一个特定的位置

就像前面的例子一样，你也可以从mysql二进制日志中截止到一个特定位置的条目，如下所示。

```
$ mysqlbinlog --stop-position=15028 mysqld-bin.000001 > upto-15028.out
```
上面的示例将在15028的位置上停止binlog。当在命令行中指定多个二进制日志文件时，停止位置将仅应用于给定列表中的最后一个二进制日志文件。

11 刷新日志以清除Binlog输出

当二进制日志文件没有被正确地关闭时，将在输出中看到一个警告消息，如下所示。

```
$ mysqlbinlog mysqld-bin.000001 > output.out
```
如下所示，报告中提示binlog文件没有正确地关闭。

```
# head output.log
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
.. ..
# Warning: this binlog is either in use or was not closed properly.
..
.. .. BINLOG ' IeZ4WQ8BAAAAZgAAAGoAAAABAAQANS4xLjczLWxvZwAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAh5nhZEzgNAAgAEgAEBAQEEgAAUwAEGggAAAAICAgC
```
当看到这个提示时，需要连接到mysql并刷新日志，如下所示。

```
mysql> flush logs;
```

刷新日志之后，再次执行mysqlbinlog命令，将不会看到在mysqlbinlog输出中binlog未正确关闭的警告消息。

12 在输出中只显示语句

默认情况下，正如在前面的示例输出中看到的一样，除了SQL语句之外，在mysqlbinlog输出中还会有一些附加信息。如果只想查看常规的SQL语句，而不需要其他内容，那么可以使用 -s 选项，如下所示。
也可以使用 --short-form 选项，效果相同。

```
$ mysqlbinlog -s mysqld-bin.000001
 
$ mysqlbinlog --short-form mysqld-bin.000001
```
下面是上述命令的部分输出。在这里，它将只显示来自给定二进制日志文件的SQL语句。

```
SET TIMESTAMP=1501096106/*!*/;
insert into employee values(400,'Nisha','Marketing',9500)/*!*/;
SET TIMESTAMP=1501096106/*!*/;
insert into employee values(500,'Randy','Technology',6000) 
.. 
..
```
不会显示像下面这样的条目：
```
# at 1201
#170726 15:08:26 server id 1  end_log_pos 1329  Query   thread_id=3     exec_time=0     error_code=0
```
13 查看特定开始时间的条目

下面将只提取从指定时间开始的条目。在此之前的任何条目都将被忽略。
```
$ mysqlbinlog --start-datetime="2017-08-16 10:00:00" mysqld-bin.000001
```
当你想要从一个二进制文件中提取数据时，这是非常有用的，因为你希望使用它来恢复或重构在某个时间段内发生的某些数据库活动。时间戳的格式可以是MySQL服务器所理解的DATETIME和timestamp中的任何类型。

14 查看特定结束时间的条目

与前面的开始时间示例一样，这里也可以指定结束时间，如下所示。

```
$ mysqlbinlog --stop-datetime="2017-08-16 15:00:00" mysqld-bin.000001
```
上面的命令将读取到给定结束时间的条目。任何来自于超过给定结束时间的mysql二进制日志文件的条目都不会被处理。

15 从远程服务器获取二进制日志

在本地机器上，还可以读取位于远程服务器上的mysql二进制日志文件。为此，需要指定远程服务器的ip地址、用户名和密码，如下所示。
此处使用-R选项。-R选项与-read-from-remote-server相同。

```
$ mysqlbinlog -R -h 192.168.101.2 -p mysqld-bin.000001

```
在上面命令中：
-R 选项指示mysqlbinlog命令从远程服务器读取日志文件
-h 指定远程服务器的ip地址
-p 将提示输入密码。默认情况下，它将使用“root”作为用户名。也可以使用 -u 选项指定用户名。
mysqld-bin.000001 这是在这里读到的远程服务器的二进制日志文件的名称。
下面命令与上面的命令完全相同：

```
$ mysqlbinlog --read-from-remote-server --host=192.168.101.2 -p mysqld-bin.000001
```
如果只指定 -h 选项，将会得到下面的错误消息。
```
$ mysqlbinlog -h 192.168.101.2 mysqld-bin.000001
mysqlbinlog: File 'mysqld-bin.000001' not found (Errcode: 2)
```
当你在远程数据库上没有足够的特权时，将得到以下“不允许连接”错误消息。在这种情况下，确保在远程数据库上为本地客户机授予适当的特权。

```
$ mysqlbinlog -R --host=192.168.101.2 mysqld-bin.000001
ERROR: Failed on connect: Host '216.172.166.27' is not allowed to connect
to this MySQL server
```
如果没有使用 -p 选项指定正确的密码，那么将得到以下“访问拒绝”错误消息。
```
$ mysqlbinlog -R --host=192.168.101.2 mysqld-bin.000001
ERROR: Failed on connect: Access denied for user 'root'@'216.172.166.27' (using password: YES)

```

下面的示例显示，还可以使用-u选项指定mysqlbinlog应该用于连接到远程MySQL数据库的用户名。请注意，这个用户是mysql用户（不是Linux服务器用户）。
```
$ mysqlbinlog -R --host=192.168.101.2 -u root -p mysqld-bin.000001<span style="text-indent: 2em;"> </span>
```
