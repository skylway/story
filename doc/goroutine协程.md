### goroutine协程

#### 一，基本使用
- 设置goroutine运行的CPU数量，最新版本的go已经默认已经设置了。
```
num := runtime.NumCPU()    //获取主机的逻辑CPU个数
runtime.GOMAXPROCS(num)    //设置可同时执行的最大CPU数
```

```
package main

import (
    "fmt"
    "time"
)

func cal(a int , b int )  {
    c := a+b
    fmt.Printf("%d + %d = %d\n",a,b,c)
}

func main() {
　　
    for i :=0 ; i<10 ;i++{
        go cal(i,i+1)  //启动10个goroutine 来计算
    }
    time.Sleep(time.Second * 2) // sleep作用是为了等待所有任务完成
} 
//结果
//8 + 9 = 17
//9 + 10 = 19
//4 + 5 = 9
//5 + 6 = 11
//0 + 1 = 1
//1 + 2 = 3
//2 + 3 = 5
//3 + 4 = 7
//7 + 8 = 15
//6 + 7 = 13
```



#### 二，goroutine异常捕捉
- 当启动多个goroutine时，如果其中一个goroutine异常了，并且我们并没有对进行异常处理，那么整个程序都会终止，所以我们在编写程序时候最好每个goroutine所运行的函数都做异常处理，异常处理采用recover

```
package main

import (
    "fmt"
    "time"
)

func addele(a []int ,i int)  {
    defer func() {    //匿名函数捕获错误
        err := recover()
        if err != nil {
            fmt.Println("add ele fail")
        }
    }()
   a[i]=i
   fmt.Println(a)
}

func main() {
    Arry := make([]int,4)
    for i :=0 ; i<10 ;i++{
        go addele(Arry,i)
    }
    time.Sleep(time.Second * 2)
}
//结果
add ele fail
[0 0 0 0]
[0 1 0 0]
[0 1 2 0]
[0 1 2 3]
add ele fail
add ele fail
add ele fail
add ele fail
add ele fail
```



#### 四，同步的goroutine
- 由于goroutine是异步执行的，那很有可能出现主程序退出时还有goroutine没有执行完，此时goroutine也会跟着退出。此时如果想等到所有goroutine任务执行完毕才退出，go提供了sync包和channel来解决同步问题，当然如果你能预测每个goroutine执行的时间，你还可以通过time.Sleep方式等待所有的groutine执行完成以后在退出程序(如上面的列子)。

- 1,示例一：使用sync包同步goroutine sync大致实现方式：WaitGroup 等待一组goroutinue执行完毕. 主程序调用 Add 添加等待的goroutinue数量. 每个goroutinue在执行结束时调用 Done ，此时等待队列数量减1.，主程序通过Wait阻塞，直到等待队列为0.

```
package main

import (
    "fmt"
    "sync"
)

func cal(a int , b int ,n *sync.WaitGroup)  {
    c := a+b
    fmt.Printf("%d + %d = %d\n",a,b,c)
    defer n.Done() //goroutinue完成后, WaitGroup的计数-1

}

func main() {
    var go_sync sync.WaitGroup //声明一个WaitGroup变量
    for i :=0 ; i<10 ;i++{
        go_sync.Add(1) // WaitGroup的计数加1
        go cal(i,i+1,&go_sync)  
    }
    go_sync.Wait()  //等待所有goroutine执行完毕
}
//结果
+ 10 = 19
+ 3 = 5
+ 4 = 7
+ 5 = 9
+ 6 = 11
+ 2 = 3
+ 7 = 13
+ 8 = 15
+ 1 = 1
+ 9 = 17
```
- 示例二：通过channel实现goroutine之间的同步。
- 实现方式：通过channel能在多个groutine之间通讯，当一个goroutine完成时候向channel发送退出信号,等所有goroutine退出时候，利用for循环channe去channel中的信号，若取不到数据会阻塞原理，等待所有goroutine执行完毕，使用该方法有个前提是你已经知道了你启动了多少个goroutine。

```
package main

import (
    "fmt"
    "time"
)

func cal(a int , b int ,Exitchan chan bool)  {
    c := a+b
    fmt.Printf("%d + %d = %d\n",a,b,c)
    time.Sleep(time.Second*2)
    Exitchan <- true
}

func main() {

    Exitchan := make(chan bool,10)  //声明并分配管道内存
    for i :=0 ; i<10 ;i++{
        go cal(i,i+1,Exitchan)
    }
    for j :=0; j<10; j++{   
         <- Exitchan  //取信号数据，如果取不到则会阻塞
    }
    close(Exitchan) // 关闭管道
}
```



