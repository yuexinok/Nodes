# time包

## 1）定时器Timer

包含两个方法，Rest() 重置  Stop停止

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t := time.NewTimer(2 * time.Second)
	now := time.Now()
	fmt.Printf("Now time : %v\n", now)

	expire := <-t.C
	
	fmt.Printf("过期时间:%v.\n", expire)
}

```

上面的实现类似sleep

```go
fmt.Printf("当前时间：%v\n", time.Now())
time.Sleep(2 * time.Second)
fmt.Printf("阻塞等待2秒:%v\n", time.Now())
```

### time.After函数

类似多少时间之后执行，**非阻塞的**。

```go
time.After(10 * time.Second)
fmt.Println("TT")//无意义
```

```Go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan int, 1)
	ch2 := make(chan int, 1)
	
	
	select {
	case e1 := <-ch1:
		fmt.Printf("CH1:%v\n", e1)
	case e2 := <-ch2:
		fmt.Printf("CH2:%v\n", e2)
	case <-time.After(10 * time.Second):
		fmt.Println("过期了！")
	}
}
//因为ch1,ch2没有写入，阻塞了，
```

### time.AfterFunc

```go
package main

import (
	"fmt"
	"time"
)
func main() {
	var t *time.Timer
	f := func() {
		fmt.Printf("定时执行了%v.\n", time.Now())
		fmt.Printf("C`s len:%d\n", len(t.C))
	}
	t = time.AfterFunc(2*time.Second, f)
	fmt.Printf("先被执行:%v\n", time.Now())
	time.Sleep(3 * time.Second)
	fmt.Printf("sleep之后:%v\n", time.Now())
}
//先被执行:2017-12-19 21:36:32.392773 +0800 CST m=+0.000460165
//定时执行了2017-12-19 21:36:34.393369 +0800 CST m=+2.001037968.
//C`s len:0
//sleep之后:2017-12-19 21:36:35.393758 +0800 CST m=+3.001418655
```

## 2）断续器

就是每隔多少时间执行

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	var ticker *time.Ticker = time.NewTicker(1 * time.Second)

	go func() {
		for t := range ticker.C {
			fmt.Println("Tick at ", t)
		}
	}()

	time.Sleep(time.Second * 5)
  //手动停止
	ticker.Stop()
	fmt.Println("Ticker stopped")
}
//Tick at  2017-12-19 21:40:57.316935 +0800 CST m=+1.000514968
//Tick at  2017-12-19 21:40:58.31699 +0800 CST m=+2.000560979
//Tick at  2017-12-19 21:40:59.317191 +0800 CST m=+3.000753613
//Tick at  2017-12-19 21:41:00.317446 +0800 CST m=+4.000999587
//Ticker stopped
//Tick at  2017-12-19 21:41:01.317891 +0800 CST m=+5.001434145
```

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	f1 := func() {
		fmt.Println("T1每隔1秒执行")
	}
	t1 := SetInterval(f1, 1*time.Second)

	//定时5秒后结束
	SetTimeout(func() {
		fmt.Println("4秒后执行")
	}, 4*time.Second)

	//阻塞5秒 模拟程序
	time.Sleep(5 * time.Second)
	//手动停止
	t1.Stop()

}

func SetInterval(f func(), t time.Duration) (ticker *time.Ticker) {
	ticker = time.NewTicker(t)
	go func() {
		for range ticker.C {
			f()
		}
	}()
	return ticker
}

func SetTimeout(f func(), t time.Duration) (timer *time.Timer) {
	//直接用afterfunc
	timer = time.AfterFunc(t, f)
	return timer
}
//T1每隔1秒执行
//T1每隔1秒执行
//T1每隔1秒执行
//T1每隔1秒执行
//4秒后执行
//T1每隔1秒执行
```

### 模拟javascript

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	f1 := func() {
		fmt.Println("T1每隔1秒执行")
	}
	t1 := SetInterval(f1, 1*time.Second)

  	//创建无缓冲通道阻塞
	var ok = make(chan bool)
	//定时5秒后结束
	SetTimeout(func() {
		fmt.Println("5秒后执行")
		//停止定时
		t1.Stop()
		//设置完成
		ok <- true
	}, 15*time.Second)
   //阻塞等待读取
	<-ok

}

func SetInterval(f func(), t time.Duration) (ticker *time.Ticker) {
	ticker = time.NewTicker(t)
   //启动一个线程
	go func() {
      	////死循环从ticker里面读取数据，读取到数据后
		for t := range ticker.C {
			fmt.Printf("当前时间：%v\n", t)
			f()
		}
	}()
	return ticker
}

func SetTimeout(f func(), t time.Duration) (timer *time.Timer) {
	//直接用afterfunc
	timer = time.AfterFunc(t, f)
	return timer
}

```

