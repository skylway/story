## php生命周期


####问问你自己，php生命周期这幅图你能理解多少？？
```
1 模块初始化阶段
2 请求初始化阶段
3 执行PHP脚本阶段
4 请求结束阶段
5 模块关闭阶段
```
![php.png][1]
####ZEND引擎编译过程
#####从最初我们编写的PHP脚本->到最后脚本被执行->得到执行结果,这个过程,可以分为如下几个阶段
```
* 首先,Zend Engine(ZE),调用词法分析器,将我们要执行的PHP源文件,去掉空格 ,注释,分割成一个一个的token.
* 然后,ZE会将得到的token forward给语法分析器,生成抽象语法树.
* 然后,ZE调用zend_compile_top_stmt()函数将抽象语法树解析为一个一个的op code,opcode一般会以op array的形式存在,它是PHP执行的中间语言.
* 最后,ZE调用zend_executor来执行op array,输出结果.
```

![1-4PHP编译执行过程.png][2]


  [1]: https://www.skylway.com/usr/uploads/2018/11/3177336532.png
  [2]: https://www.skylway.com/usr/uploads/2018/11/3259568662.png