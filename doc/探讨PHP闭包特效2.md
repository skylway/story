## 探讨PHP闭包特效2

###改变闭包use上下文的变量
use所引用的也只不过是变量的一个副本而已。但是我想要完全引用变量，而不是复制。
要达到这种效果，其实在变量前加一个 & 符号就可以了：

```
function getMoney() {
    $rmb = 1;
    $func = function() use ( &$rmb ) {
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
//2
```
好，这样匿名函数就可以引用上下文的变量了。如果将匿名函数返回给外界，匿名函数会保存use所引用的变量，而外界则不能得到这些变量，这样形成‘闭包’这个概念可能会更清晰一些。

根据描述改变一下上面的例子：

```
function getMoneyFunc() {
    $rmb = 1;
    $func = function() use ( &$rmb ) {
        echo $rmb;
        //把$rmb的值加1
        $rmb++;
    };
    return $func;
}

$getMoney = getMoneyFunc();
$getMoney();
$getMoney();
$getMoney();

//输出：
//1
//2
//3
```

###array_map闭包
我们通常把PHP闭包当做函数会方法的回调使用，事实上，很多PHP函数都会用到闭包，比如array_map和preg_replace_callback，这是使用PHP匿名函数的绝佳时机。记住，闭包和其他值一样，可以作为参数传入其他PHP函数：
```
$numberPlusOne = array_map(function ($number) {
    return $number += 1;
}, [1, 2, 3]);

print_r($numberPlusOne);
```
在闭包出现之前，要实现这样的功能，PHP开发者只能单独创建具名函数，然后使用名称引用这个函数：

```
function incrementNumber ($number) {
    return $number += 1;
}

$numberPlusOne = array_map(‘incrementNumber’,  [1, 2, 3]);
print_r($numberPlusOne);
```

###闭包的好处

#####1 减少foreach的循环的代码
#####2 减少函数的参数
#####3 解除递归函数

```
    $fib =function($n)use(&$fib) {
        if($n == 0 || $n == 1) return 1;
        return $fib($n - 1) + $fib($n - 2);
    };
 
   echo $fib(2) . "\n";// 2
   $lie =$fib;
   $fib =function(){die('error');};//rewrite $fib variable 
   echo $lie(5);// error   because $fib is referenced by closure
```
注意上题中的use使用了&，这里不使用&会出现错误n-1)是找不到function的（前面没有定义fib的类型）

所以想使用闭包解除循环函数的时候就需要使用
```
$recursive =function ()use (&$recursive){
// The function is now available as $recursive
}
```
#####4 关于延迟绑定
如果你需要延迟绑定use里面的变量，你就需要使用引用，否则在定义的时候就会做一份拷贝放到use中

```
$result = 0;
 
$one =function()
{ var_dump($result); };
 
$two =function()use ($result)
{ var_dump($result); };
 
$three =function()use (&$result)
{ var_dump($result); };
 
$result++;
 
$one();   // outputs NULL: $result is not in scope
$two();   // outputs int(0): $result was copied
$three();   // outputs int(1)
```
使用引用和不使用引用就代表了是调用时赋值，还是申明时候赋值



