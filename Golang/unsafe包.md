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

