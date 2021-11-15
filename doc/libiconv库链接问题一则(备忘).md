## libiconv库链接问题一则(备忘)

与在Solaris系统上不同，Linux的libc库中包含了libiconv库中函数的定义，因此在Linux上使用libiconv库相关函数，编译时是不需要显式-liconv的。但最近我的一位同事在某redhat enterprise server 5.6机器上编译程序时却遇到了找不到iconv库函数符号的链接问题，到底是怎样一回事呢？这里分享一下问题查找过程。

一、现场重现

这里借用一下这位同事的测试程序以及那台机器，重现一下问题过程：
/*test.c */
```
include <iconv.h>

int main(void)
{
    int r;
    char *sin, *sout;
    size_t lenin, lenout;
    char *src = "你好!";
    char dst[256] = {0};
    iconv_t c_pt;  

    sin = src;
    lenin = strlen(src)+1;

    sout = dst;
    lenout = 256;

    if ((c_pt = iconv_open("UTF-8", "GB2312")) == (iconv_t)(-1)){
        printf("iconv_open error!. errno[%d].\n", errno);
        return -1;
    }

    if ((r = iconv(c_pt, (char **)&sin, &lenin, &sout, &lenout)) != 0){
        printf("iconv error!. errno[%d].\n", r);
        return -1;
    }  

    iconv_close(c_pt);

    printf("SRC[%s], DST[%s].\n", src, dst);

    return 0;
}
```
根据之前的经验，我们按如下命令编译该程序：
```
$> gcc -g -o test test.c

/tmp/ccyQ5blC.o: In function `main':
/home/tonybai/tmp/test.c:28: undefined reference to `libiconv_open'
/home/tonybai/tmp/test.c:33: undefined reference to `libiconv'
/home/tonybai/tmp/test.c:38: undefined reference to `libiconv_close'
```
咦，这是咋搞的呢？怎么找不到iconv库的符号！！！显式加上iconv的链接指示再试试。
```
$> gcc -g -o test test.c -liconv
```
这回编译OK了。的确如那位同事所说出现了怪异的情况。

二、现场取证

惯性思维让我首先提出疑问：难道是这台机器上的libc版本有差异，检查一下libc中是否定义了iconv相关符号。
```
$ nm /lib64/libc.so.6 |grep iconv
000000397141e040 T iconv
000000397141e1e0 T iconv_close
000000397141ddc0 T iconv_open
```
iconv的函数都定义了呀！怎么会链接不到？

我们再来看看已经编译成功的那个test到底连接到哪个iconv库了。
```
$ ldd test
    linux-vdso.so.1 =>  (0x00007fff77d6b000)
    libiconv.so.2 => /usr/local/lib/libiconv.so.2 (0x00002abbeb09e000)
    libc.so.6 => /lib64/libc.so.6 (0×0000003971400000)
    /lib64/ld-linux-x86-64.so.2 (0×0000003971000000)
```
哦，系统里居然在/usr/local/lib下面单独安装了一份libiconv。gcc显然是链接到这里的libiconv了，但gcc怎么会链接到这里了呢？

三、大侦探的分析^_^

Gcc到底做了什么呢？我们看看其verbose的输出结果。
```
$ gcc -g -o test test.c -liconv -v
使用内建 specs。
目标：x86_64-redhat-linux
配置为：../configure –prefix=/usr –mandir=/usr/share/man –infodir=/usr/share/info –enable-shared –enable-threads=posix –enable-          checking=release –with-system-zlib –enable-__cxa_atexit –disable-libunwind-exceptions –enable-libgcj-multifile –enable-languages=c,c++,   objc,obj-c++,java,fortran,ada –enable-java-awt=gtk –disable-dssi –disable-plugin –with-java-home=/usr/lib/jvm/java-1.4.2-gcj-1.4.2.0/jre –with-cpu=generic –host=x86_64-redhat-linux
线程模型：posix
gcc 版本 4.1.2 20080704 (Red Hat 4.1.2-50)
 /usr/libexec/gcc/x86_64-redhat-linux/4.1.2/cc1 -quiet -v test.c -quiet -dumpbase test.c -mtune=generic -auxbase test -g -version -o /tmp/     ccypZm0v.s
忽略不存在的目录“/usr/lib/gcc/x86_64-redhat-linux/4.1.2/../../../../x86_64-redhat-linux/include”
#include "…" 搜索从这里开始：
#include <…> 搜索从这里开始：
 /usr/local/include
 /usr/lib/gcc/x86_64-redhat-linux/4.1.2/include
 /usr/include
搜索列表结束。
GNU C 版本 4.1.2 20080704 (Red Hat 4.1.2-50) (x86_64-redhat-linux)
    由 GNU C 版本 4.1.2 20080704 (Red Hat 4.1.2-50) 编译。
GGC 准则：–param ggc-min-expand=100 –param ggc-min-heapsize=131072
Compiler executable checksum: ef754737661c9c384f73674bd4e06594
 as -V -Qy -o /tmp/ccaqvDgX.o /tmp/ccypZm0v.s
GNU assembler version 2.17.50.0.6-14.el5 (x86_64-redhat-linux) using BFD version 2.17.50.0.6-14.el5 20061020
 /usr/libexec/gcc/x86_64-redhat-linux/4.1.2/collect2 –eh-frame-hdr -m elf_x86_64 –hash-style=gnu -dynamic-linker /lib64/ld-linux-x86-64.so.  2 -o test /usr/lib/gcc/x86_64-redhat-linux/4.1.2/../../../../lib64/crt1.o /usr/lib/gcc/x86_64-redhat-linux/4.1.2/../../../../lib64/crti.o /usr/   lib/gcc/x86_64-redhat-linux/4.1.2/crtbegin.o -L/usr/lib/gcc/x86_64-redhat-linux/4.1.2 -L/usr/lib/gcc/x86_64-redhat-linux/4.1.2 -L/usr/lib/gcc/ x86_64-redhat-linux/4.1.2/../../../../lib64 -L/lib/../lib64
-L/usr/lib/../lib64 /tmp/ccaqvDgX.o -liconv -lgcc –as-needed -lgcc_s –no-as-needed -lc -lgcc –as-needed -lgcc_s –no-as-needed /usr/lib/gcc/x86_64-redhat-linux/4.1.2/crtend.o /usr/lib/gcc/x86_64-redhat-linux/4.1.2/../../../../lib64/crtn.o
```
从这个结果来看，gcc在search iconv.h这个头文件时，首先找到的是/usr/local/include/iconv.h，而不是/usr/include/iconv.h。这两个文件有啥不同么？

在/usr/local/include/iconv.h中，我找到如下代码：
```
…
#ifndef LIBICONV_PLUG
#define iconv_open libiconv_open
#endif
extern iconv_t iconv_open (const char* tocode, const char* fromcode);
…

libiconv_open vs iconv_open，卧槽！！！再对比一下前面编译时输出的错误信息：

/tmp/ccyQ5blC.o: In function `main':
/home/tonybai/tmp/test.c:28: undefined reference to `libiconv_open'
/home/tonybai/tmp/test.c:33: undefined reference to `libiconv'
/home/tonybai/tmp/test.c:38: undefined reference to `libiconv_close'
```
大侦探醒悟了！大侦探带你还原一下真实情况。

我们在执行gcc -g -o test test.c时， 根据gcc -v中include search dir的顺序，gcc首先search到的是/usr/local/include/iconv.h，而这里iconv_open等函数被预编译器替换成 了libiconv_open等加上了lib前缀的函数，而这些函数符号显然在libc中是无法找到的，libc中只有不带lib前缀的 iconv_open等函数的定义。大侦探也是一时眼拙了，没有细致查看gcc的编译错误信息中的内容，这就是问题所在！

而gcc -g -o test test.c -liconv为何可以顺利编译通过呢？gcc是如何找到/usr/local/lib下的libiconv的呢？大侦探再次为大家还原一下真相。

我们在执行gcc -g -o test test.c -liconv时，gcc同 样首先search到的是/usr/local/include/iconv.h，然后编译test.c源码，ok；接下来启动ld程序进行链接；ld找 到了libiconv，ld是怎么找到iconv的呢，libiconv在/usr/local/lib下，ld显然是到这个目录下search了。我们 通过执行下面命令可以知晓ld的默认搜索路径：
```
$> ld -verbose|grep SEARCH
SEARCH_DIR("/usr/x86_64-redhat-linux/lib64"); SEARCH_DIR("/usr/local/lib64"); SEARCH_DIR("/lib64"); SEARCH_DIR("/usr/lib64"); SEARCH_DIR("/usr/x86_64-redhat-linux/lib"); SEARCH_DIR("/usr/lib64"); SEARCH_DIR("/usr/local/lib"); SEARCH_DIR("/lib"); SEARCH_DIR("/usr/lib");
```
ld的默认search路径中有/usr/local/lib(我之前一直是以为/usr/local/lib不是gcc/ld的默认搜索路径的)，因此找到libiconv就不足为奇了。

```
方案一：
可以将/usr/local/lib下的那份libiconv卸载掉！
方案二：
gcc 加上 -liconv
方案三：
make编译强制使用libc中的iconv实现了，CCFLAGS = -DLIBICONV_PLUG
```