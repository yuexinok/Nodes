### 关键字：

break	default	func	interface	select

case	defer	go	map	struct

chan	else	goto	package	switch

const	fallthrough	if	range	type

continue  for	import return var

### 内常量：

true	false	 iota	nil

### 内函数：

make

len

cap

new

append

copy

close

delete

complex

real

imag

panic

reconver

### 变量

```Go
x:= 1
&x //取地址
*x //读取地址指向的变量值
//任何指针的零值都为nil
var x,y int
fmt.Println(&x == &x,&x == &y,&x == nil)//true false false

p := new(int) //p,*int类型
fmt.Println(*p) //0
func newInt() *int {
	return new(int)
}
func newInt() *int {
	var dummy int
	return &dummy
}

p := new(int)
	fmt.Println(p)//0xc42000e240
	fmt.Println(*p)//0
	fmt.Println(&p)//0xc42000c028


	//p = 2//错误 cannot use 2 (type int) as type *int in assignment
	//fmt.Println(p)
	//fmt.Println(p)
	//fmt.Println(*p)
	//fmt.Println(&p)
	*p = 3
	fmt.Println(p) //0xc42000e240
	fmt.Println(*p)//3
	fmt.Println(&p) //0xc42000c028
	//&p = 4 //cannot assign to &p,cannot use 4 (type int) as type **int in assignment
	//fmt.Println(p)
	//fmt.Println(*p)
	//fmt.Println(&p)
	s := &p
	//s = 212//cannot use 212 (type int) as type **int in assignment
	//*s = 212 //cannot use 212 (type int) as type *int in assignment
	fmt.Println(s)
	a := p
	//a = 213 //cannot use 213 (type int) as type *int in assignment
	*a = 213
	fmt.Println(p)
	fmt.Println(*p)
	fmt.Println(&p)

	fmt.Println(a)
	fmt.Println(*a)
	fmt.Println(&a)
```

```go
//&p 取地址 ，得到只是一个地址。
//*p 是针对 指针的()，取指针(地址)的值，赋值指针的值
```

### 赋值

#### 元组赋值

x,y = y,x

a[i],a[j] = a[j],a[i]

#### 普通

`a++` 为语句不是，表达式

所以不能

x := i++

```go
v = m[key]                // map查找，失败时返回零值
v = x.(T)                 // type断言，失败时panic异常
v = <-ch                  // 管道接收，失败时返回零值（阻塞不算是失败）

_, ok = m[key]            // map返回2个值
_, ok = mm[""], false     // map返回1个值
_ = mm[""]                // map返回1个值
```

### 类型

type 类型名称 底层类型名称

```go
type Celsius float64
type Fah float64
const (
	Absso Celsius = -2123.2
  	Free Fah = 213
)
```