## 探讨PHP闭包特效1

###闭包概念

提到闭包就不得不想起匿名函数，也叫闭包函数（closures），貌似PHP闭包实现主要就是靠它。声明一个匿名函数是这样：
```
$func = function() {
    
}; //带结束符
```
可以看到，匿名函数因为没有名字，如果要使用它，需要将其返回给一个变量。匿名函数也像普通函数一样可以声明参数，调用方法也相同：

```
$func = function( $param ) {
    echo $param;
};

$func( 'some string' );

//输出：
//some string
```
###实现闭包
将匿名函数在普通函数中当做参数传入，也可以被返回。这就实现了一个简单的闭包。

下边有例子：
```
function getPrintStrFunc() {
    $func = function( $str ) {
        echo $str;
    };
    return $func;
}

$printStrFunc = getPrintStrFunc();
$printStrFunc( 'some string' );
```
```
function callFunc( $func ) {
    $func( 'some string' );
}

$printStrFunc = function( $str ) {
    echo $str;
};
callFunc( $printStrFunc );
```

###连接闭包和外界变量的关键字：USE
闭包可以保存所在代码块上下文的一些变量和值。PHP在默认情况下，匿名函数不能调用所在代码块的上下文变量，而需要通过使用use关键字。

```
function getMoney() {
    $rmb = 1;
    $dollar = 6;
    $func = function() use ( $rmb ) {
        echo $rmb;
        echo $dollar;
    };
    $func();
}

getMoney();

//输出：
//1
//报错，找不到dorllar变量
```
可以看到，dollar没有在use关键字中声明，在这个匿名函数里也就不能获取到它，所以开发中要注意这个问题。

 

有人可能会想到，是否可以在匿名函数中改变上下文的变量，但我发现是不可以的：

```
function getMoney() {
    $rmb = 1;
    $func = function() use ( $rmb ) {
        echo $rmb;
        //把$rmb的值加1
        $rmb++;
    };
    $func();
    echo $rmb;
}

getMoney();

//输出：
//1
//1
```
