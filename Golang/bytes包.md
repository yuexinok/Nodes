# bytes包：

## 方法：

```go
func ToUpper(s []byte) []byte
func ToLower(s []byte) []byte
func ToTitle(s []byte) []byte
//使用映射表
func ToUpperSpecial(_case unicode.SpecialCase,s []byte) []byte

func Title(s []byte) []byte
//比较 a<b -1 a==b 0
func Compare(a,b []byte) int
//是否相等
func Equal(a,b []byte) bool
//是否相似 忽略大小写和标题
func EqualFold(s,t []byte) bool
//去掉包含在cutset中的字符
func Trim(s []byte,cutset string) []byte
func TrimLeft(s []byte,cutset string) []byte
func TrimRight(s []byte,cutset string) []byte
func TrimFunc(s []byte,f func(r rune) bool) []byte
//去掉空白
func TrimSpace(s []byte) []byte
//去掉前缀prefix
func TrimPrefix(s,prefix []byte) []byte
func TrimSuffix(s,prefix []byte) []byte
```

```go
s1 := "Φφϕ kKK 中国"
s2 := "ϕΦφ KkK 中国"

for _, c := range s1 {
  fmt.Printf("%-5x", c) //3a6  3c6  3d5  20   6b   4b   212a 20   4e2d 56fd
}
fmt.Println()

for _, c := range s2 {
  fmt.Printf("%-5x", c) //3d5  3a6  3c6  20   212a 6b   4b   20   4e2d 56fd
}
fmt.Println()

fmt.Println(bytes.EqualFold([]byte(s1), []byte(s2))) //true

bs := [][]byte{
  []byte("Hello World！"),
  []byte("Hello 时间！"),
  []byte("Hello golang."),
}
f := func(r rune) bool {
  return bytes.ContainsRune([]byte("!！. "), r)
}
for _, b := range bs {
  fmt.Printf("%q\n", bytes.TrimFunc(b, f))
}
//"Hello World"
//"Hello 时间"
//"Hello golang"
for _, b := range bs {
  fmt.Printf("%q\n", bytes.TrimPrefix(b, []byte("Hello")))
}
//" World！"
//" 时间！"
//" golang."
```

```go
//拆合
//以sep为分割符号
func Split(s,sep []byte) [][]byte
//n限制个数
func SplitN(s,sep []byte,n int) [][]byte
//同Split，只不过结果包含分割符
func SplitAfter(s,sep []byte) [][]byte
func SplitAfterN(s,sep []byte,n int) [][]byte
//以连续空白为分割符号，将s切分成多个子串
func Fields(s []byte) [][]byte
func FieldsFunc(s []byte,f func(rune) bool) [][]byte
func Join(s [][]byte,sep []byte) []byte
func Repeat(b []byte,count int)[]byte

b1 := []byte(" Hello  World! ")
fmt.Printf("%q\n", bytes.Split(b1, []byte{' '}))
//["" "Hello" "" "World!" ""]
fmt.Printf("%q\n", bytes.Fields(b1))
//["Hello" "World!"]
f1 := func(r rune) bool {
  return bytes.ContainsRune([]byte("!"), r)
}
fmt.Printf("%q\n", bytes.FieldsFunc(b1, f1))
//[" Hello  World" " "]
```



```go
func HasPrefix(s,prefix []byte) bool
func HasSuffix(s,suffix []byte) bool

//是否包含
func Contains(b,subslice []byte) bool
func ContainsRune(b []byte,r rune) bool
//判断b中是否包含chars中的任何一个字符
func ContainsAny(b []byte,chars string) bool
//查找子串sep
func Index(s,sep []byte) int
func IndexByte(s []byte,c byte) int
func IndexRune(s []byte,r rune) int
func IndexAny(s []byte,chars string) int
func IndexFunc(s []buyte,f func(r rune) bool) int

func LastIndex(s,sep []byte) int
func LastIndexByte(s []byte,c byte) int
func LastIndexAny(s []byte,chars string) int
func LastIndexFunc(s []byte,f func(r rune) bool) int

func Replace(s,old,new []byte,n int) []byte
func Map(mapping func(r rune) rune,s []byte) []byte
func Runnes(s []buyte) []rune

//把byte封装成Reader对象
func NewReader(b []byte) *Reader
//返回未读取部分的数据长度
func (r *Reader) Len() int
func (r *Reader) Size() int64
func (r *Reader) Reset(b []byte)
```



```Go
b2 := [][]byte{
[]byte("Hello Wolrd!"),
[]byte("Hello 世界！"),
}
buf := make([]byte, 6)
rd := bytes.NewReader(b2[0])
rd.Read(buf)
fmt.Printf("%q\n", buf) //"Hello "
rd.Read(buf)
fmt.Printf("%q\n", buf) //"Wolrd!"

rd.Reset(b2[1])
rd.Read(buf)
fmt.Printf("%q\n", buf)                             //"Hello "
fmt.Printf("Size:%d,Len:%d\n", rd.Size(), rd.Len()) //Size:15,Len:9
```

```go
//创建Buffer对象
func NewBuffer(buf []byte) *Buffer
func NewBufferString(s string) *Buffer
func (b *Buffer) Len() int
func (b *Buffer) Cap() int
func (b *Buffer) Next(n int) []byte
func (b *Buffer) ReadBytes(delim byte) (line []byte,err error)
func (b *Buffer) WriteRune(r rune) (n int,err error)
func (b *Buffer) Reset()
```

```go
rd1 := bytes.NewBufferString("Hello World!")
buf1 := make([]byte, 6)
b := rd1.Bytes()
rd1.Read(buf1)
fmt.Printf("%s\n", rd1.String()) //World!
fmt.Printf("%s\n", b)            //Hello World!
rd1.Write([]byte("abcdefg"))
fmt.Printf("%s\n", rd1.String()) //World!abcdefg
fmt.Printf("%s\n", b)            //Hello World!

rd1.Read(buf1)                   //继续读取
fmt.Printf("%s\n", rd1.String()) //abcdefg
fmt.Printf("%s\n", b)            //Hello World!
```