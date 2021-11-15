## php迭代生成器

###一，先看看生成器类PHP-Iterator迭代器(遍历)接口例子：
####类摘要 

```
Generator implements Iterator {
/* 方法 */
public mixed current ( void )
public mixed key ( void )
public void next ( void )
public void rewind ( void )
public mixed send ( mixed $value )
public void throw ( Exception $exception )
public bool valid ( void )
public void __wakeup ( void )
}

Generator::current — 返回当前产生的值
Generator::key — 返回当前产生的键
Generator::next — 生成器继续执行
Generator::rewind — 重置迭代器
Generator::send — 向生成器中传入一个值
Generator::throw — 向生成器中抛入一个异常
Generator::valid — 检查迭代器是否被关闭
Generator::__wakeup — 序列化回调
```

####测试用例：

```
<?php
class Number implements Iterator{  
    protected $i = 1;
    protected $key;
    protected $val;
    protected $count; 
    public function __construct(int $count){
        $this->count = $count;
        echo "第{$this->i}步:对象初始化.\n";
        $this->i++;
    }
    public function rewind(){
        $this->key = 0;
        $this->val = 0;
        echo "第{$this->i}步:rewind()被调用.\n";
        $this->i++;
    }
    public function next(){
        $this->key += 1;
        $this->val += 2;
        echo "第{$this->i}步:next()被调用.\n";
        $this->i++;
    }
    public function current(){
        echo "第{$this->i}步:current()被调用.\n";
        $this->i++;
        return $this->val;
    }
    public function key(){
        echo "第{$this->i}步:key()被调用.\n";
        $this->i++;
        return $this->key;
    }
    public function valid(){
        echo "第{$this->i}步:valid()被调用.\n";
        $this->i++;
        return $this->key < $this->count;
    }
}

$number = new Number(5);
echo "start...\n";
foreach ($number as $key => $value){
echo "{$key} - {$value}\n";
}
echo "...end...\n";
```

####查看结果如下：流程大概为 rewind初始化迭代器->valid检查是否关闭迭代器->current返回迭代器当前产生值->key返回当前键->next跌打器继续执行->valid->current->key一直到valid判断结果结束

```
第1步:对象初始化.
start...
第2步:rewind()被调用.
第3步:valid()被调用.
第4步:current()被调用.
第5步:key()被调用.
0 - 0
第6步:next()被调用.
第7步:valid()被调用.
第8步:current()被调用.
第9步:key()被调用.
1 - 2
第10步:next()被调用.
第11步:valid()被调用.
第12步:current()被调用.
第13步:key()被调用.
2 - 4
第14步:next()被调用.
第15步:valid()被调用.
第16步:current()被调用.
第17步:key()被调用.
3 - 6
第18步:next()被调用.
第19步:valid()被调用.
第20步:current()被调用.
第21步:key()被调用.
4 - 8
第22步:next()被调用.
第23步:valid()被调用.
...end...
```

###二，接下来看看yield生成器
####yield关键字实现的功能与Generator类似,yield只能在函数内使用,每一次遇到yield返回就是当前跌迭代，当程序运行到yield的时候，当前程序就唤起协程记录上下文，然后主函数继续操作，当需要操作的时候，在通过迭代器的next重新调起

例子一：
```
function test()
 {
        $tt = 10000000;
        for ($i = 1;$i <= $tt; $i++) {
            yield $i;
        }
}

$a=test();
$i = 0;
foreach ($a as $key => $value) {
    echo "{$key} - {$value}\n";
}
```
```
0 - 1
1 - 2
2 - 3
3 - 4
4 - 5
5 - 6
6 - 7
7 - 8
8 - 9
9 - 10
......
```
例子二：
```
<?php
function gen() {
    $ret = (yield 'yield1');
    var_dump($ret);
    $ret = (yield 'yield2');
    var_dump($ret);
}
 
$gen = gen();
var_dump($gen->current());    // string(6) "yield1"
var_dump($gen->send('ret1')); // string(4) "ret1"   (the first var_dump in gen)
                              // string(6) "yield2" (the var_dump of the ->send() return value)
var_dump($gen->send('ret2')); // string(4) "ret2"   (again from within gen)
                              // NULL               (the return value of ->send())
?>
```

```
string(6) "yield1"
string(4) "ret1"
string(6) "yield2"
string(4) "ret2"
NULL
```

