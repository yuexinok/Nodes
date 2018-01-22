# 日期时间处理：

## 1）获取时间戳：

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println(time.Now())        //2018-01-02 15:47:36.728073 +0800 CST m=+0.000484143
	timestamp := time.Now().Unix() //1514879256
	fmt.Println(timestamp)
	fmt.Println(time.Now().UnixNano()) //1514879299047858000
}
```

## 2）时间戳转日期：

```go
var timeunix int64 = 1514779200 //int64
tm := time.Unix(timeunix, 0) //转换为time类型
```

## 3）日期格式化：

```go
fmt.Println(tm.Format("2006年01月02日 03:04:05")) //2018年01月01日 12:00:00

//字符串转时间
tm2, _ := time.Parse("2006年01月02日", "2017年12月31日")
fmt.Println(tm2.String())//2017-12-31 00:00:00 +0000 UTC

//直接格式2017-12-31 00:00:00
fmt.Println(tm2.Format("2006-01-02 15:04:05"))
```

口诀就是2006，0102030405 PM 其实03为15对应下午3点



## 4）实例化日期:

```go
//实例化日期
tm3 := time.Date(2018, 1, 2, 16, 20, 12, 0, time.Local)
fmt.Println(tm3.Unix())                        //1514881212
fmt.Println(tm3.Format("2006-01-02 15:04:05")) //2018-01-02 16:20:12
```

