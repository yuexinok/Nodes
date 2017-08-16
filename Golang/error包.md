# error包：

## 结构：

```go
type error interface {
  Error() string
}
//error可以是任何类型，即error值实际可以包含任意信息，不一定是字符串

```

## 范例：

```go
package main

import (
	"fmt"
	"time"
)

type myError struct {
	err   string
	time  time.Time
	count int
}

func (m *myError) Error() string {
	return fmt.Sprintf("%s %d 次。时间:%v", m.err, m.count, m.time)
}
func newErr(s string, i int) *myError {
	return &myError{
		err:   s,
		time:  time.Now(),
		count: i,
	}
}

var count int

func SomeFunc() error {
	if true {
		count++
		return newErr("遇到某某情况", count)
	}
	return nil
}
func main() {
	err := SomeFunc()
	fmt.Println(err)
	//遇到某某情况 1 次。时间:2017-08-15 17:48:52.382783506 +0800 CST
}

```

