## PHP接收JSON格式的数据(备忘)

在API服务中，目前流行采用json形式来交互。
```
PHP默认识别的数据类型是application/x-www.form-urlencoded标准的数据类型。因此，对型如text/xml 或者 soap 或者 application/octet-stream 和application/json格式之类的内容无法解析，如果用$_POST数组来接收就会失败！

只有Content-Type为application/x-www-form-urlencoded或multipart/form-data的情况下，$_POST里面才有值。在其他情况下，对型如text/xml 或者 soap 或者 application/octet-stream 和application/json格式之类的内容无法解析，如果用$_POST数组来接收就会失败，$_POST里面是没有值的。

对于客户端POST提交的 Content-Type: application/json 的情况，或其他未明确指定 Content-Type 的情况，PHP可以用 file_get_contents("php://input"); 读出POST进来的数据。旧版本的php也可以用$GLOBALS['HTTP_RAW_POST_DATA']读取（php7已移除这个全局变量）
```

网上有网友还碰到了编码问题如下：
```
以下是 20180807 Liigo补记：
呵呵，有点尴尬呀。前面说“现在总算明白了”，事实证明还是有点不太明白。

后来又发现了奇怪的现象。客户端POST提交GB18030编码的带汉字的文本，PHP端用file_get_contents("php://input") 接收然后用 file_put_contents() 写出文件，用文本编辑器打开文件显示正常。而如果客户端POST提交UTF-8编码的相同内容的文本，PHP端用相同的方法接收和写出文件，用文本编辑器打开文件就是乱码。后来证明文本编辑器的编码识别没有问题，进而证明第二次写出的文件编码即不是UTF-8也不是GB18030。这就很奇怪。

我心中提出了几个问题：

file_get_contents("php://input") 视其输入数据为什么编码？其输出数据是什么编码？内部有没有进行过编码转换？
用 file_put_contents() 写出文件的过程中有没有编码转换？其输入和输出数据各是什么编码呢？
前面提到的POST提交数据时没有明确指定Content-Type ，结果是否于此有关呢？
第一次POST的编码是GB18030，写出的文件编码也是GB18030，这能说明什么呢？


目前能确认的结论是：（在上述情况下）POST只能提交GB18030编码，不能提交UTF-8编码，否则得到的结果是乱码。

在提交GB18030编码的情况下（情况1），PHP网页返回的是UTF-8编码的Respone（估计是 echo 默认输出UTF-8），转换为GB18030后，还原为原始的数据，结果是正确的。在提交UTF-8编码的情况下（情况2），PHP网页返回的数据不知道是什么编码，既不是UTF-8也不是GB18030，怎么转换结果都不对。仔细校验后发现，情况2返回的respone数据是在UTF-8编码基础上再执行UTF-8编码的结果，由此可以断定，PHP是把客户端提交的UTF-8编码的数据当作GB18030编码处理了。

客户端POST提交数据时并没有明确指定数据编码，故PHP假定其为某个编码（此处为GB18030，估计跟PHP配置有关）也有其合理性。那么POST提交时能否指定编码呢？好像是有，Content-Type: application/json; charset=utf-8 。然而好像并不管用

```