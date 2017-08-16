# unsafe包

非安全包，不建议使用

## Sizeof函数

获取类型占用的内存大小

```go
fmt.Println(unsafe.Sizeof(true))                   //1
fmt.Println(unsafe.Sizeof(int8(0)))                //1
fmt.Println(unsafe.Sizeof(int16(10)))              //2
fmt.Println(unsafe.Sizeof(int32(10000000)))        //4
fmt.Println(unsafe.Sizeof(int64(10000000000000)))  //8
fmt.Println(unsafe.Sizeof(int(10000000000000000))) //8
```

## Unsafe.Pointer

**普通指针，用于传递对象地址，不能进行指针运算**。

Unsaf.Pointer：通用指针类型，用于转换不同类型的指针，不能进行指针运算。

unitptr:用于指针运算，GC不把uintptr当指针，无法持有对象。uintptr类型目标会被回收。

Unsafe.Pointer可以和 普通指针进行互相转换。

unsafe.Pointer可以和uintptr进行相互转换。

```Go
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	s := struct {
		a byte
		b byte
		c byte
		d int64
	}{0, 0, 0, 0}

	//将机构图指针转为通用指针
	p := unsafe.Pointer(&s)
	//保存机构体的地址备用 偏移量为0
	up0 := uintptr(p)
	//将通用指针转为byte型指针
	pb := (*byte)(p)
	*pb = 10
	//结构体也变了
	fmt.Println(s) //{10 0 0 0}

	//偏移到第2个字段
	up := up0 + unsafe.Offsetof(s.b)
	p = unsafe.Pointer(up)

	pb = (*byte)(p)
	*pb = 20
	fmt.Println(s) //{10 20 0 0}
}

```

