# path包：

```go
路径分割符号
const (
	Separator = os.PathSeparator //路径分割符号
  	ListSeparator = os.PathListSeparator //路径列表分割符
)

//将path中的平台相关路径分割符号转为'/'
func ToSlash(path string) string
//将path中的'/'转为系统相关的路径分割符号


s := `http://www.bigbig.me/a/b/c/d`
u, _ := url.Parse(s)
s = u.Path
s = filepath.FromSlash(s) ///a/b/c/d
fmt.Println(s)
//在源代码下创建了目录a/b/c
if err := os.MkdirAll(s[1:], 0777); err != nil {
  fmt.Println(err)
}
s = filepath.ToSlash(s)
fmt.Println(s) ///a/b/c/d
```

```go
//获取path中最后一个分割符之前的部分(不含分割符号)
func Dir(path string) string
//获取path中最后一个分割符之后的部分
func Base(path string) string
func Split(path string) (dir,file string)
func Ext(path string) string

path := `a//b//c///d.txt`
path = filepath.FromSlash(path)
d1 := filepath.Dir(path)
f1 := filepath.Base(path)
d2, f2 := filepath.Split(path)
fmt.Printf("%q %q\n%q %q\n", d1, f1, d2, f2)
//"a/b/c" "d.txt"
//"a//b//c///" "d.txt"

ext := filepath.Ext(path)
fmt.Println(ext) //.txt
```



```Go
//获取targpath相对于basepath的路径
//targpath和basepath必须都是相对路径或者绝对路径
func Rel(basepath,targpath string) (string,error)

//都是相对
s, err := filepath.Rel(`a/b/c`, `a/b/c/d/e`)
fmt.Println(s, err) //d/e <nil>

s, err = filepath.Rel("/a/b/c", "/a/b/c/d/e")
fmt.Println(s, err) //d/e <nil>
//一个绝对 一个相对
s, err = filepath.Rel("/ab/c", "a/b/c/d/e")
fmt.Println(s, err) //Rel: can't make a/b/c/d/e relative to /ab/c
```

```go
//合并成一个路径，忽略空元素，清理多余字符
func Join(elem ...string) string
s := filepath.Join("a", "b", "", ":::", " ", `//c///d///`)
fmt.Println(s) //a/b/:::/ /c/d

//清除多余字符
func Clean(path string) string
s = filepath.Clean("a/./b/:::/..//  /c/..///d///")
fmt.Println(s) //a/b/  /d
```

```go
//获取绝对路径
func Abs(path string) (string,error)
//判断是否是绝对路径
func IsAbes(path string) bool
fmt.Println(filepath.Abs("a/b/c"))    ///Users/borgxiao/Documents/work/gotest/a/b/c <nil>
fmt.Println(filepath.IsAbs("a/b/c"))  //false
fmt.Println(filepath.IsAbs("/a/b/c")) //true
```

```go
//判断name是否指定模式完全匹配
// ? 匹配单个任意字符(不匹配路径分割符号)
// * 匹配0/n个任意字符(不匹配路径分割符号)
// []匹配范围的任意个字符(可包含路径分隔符号)
// [^]同[]反
fmt.Println(filepath.Match("???", "abc"))          //true
fmt.Println(filepath.Match("???", "abcd"))         //false
fmt.Println(filepath.Match("*", "abc"))            //true
fmt.Println(filepath.Match("*", ""))               //true
fmt.Println(filepath.Match(`???\\???`, `abc\def`)) //true
```

```go
//遍历指定目录(包含子目录) ，对遍历的项目用walkFn处理
func Walk(root string,walkFn WalkFunc) error
type WalkFunc func(path string,info os.FileInfo,err error) error


err = filepath.Walk(`.`, findTxtDir)
fmt.Println(err)
func findTxtDir(path string, info os.FileInfo, err error) error {
	ok, err := filepath.Match(`*.txt`, info.Name())
	if ok {
		fmt.Println(filepath.Dir(path), info.Name())
		return filepath.SkipDir
	}
	return err
}
```

