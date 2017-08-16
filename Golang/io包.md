# IO包：

```go
package main

import (
	"bytes"
	"errors"
	"fmt"
	"io"
	"os"
	"strings"
	"time"
)

//定义一个类型
type Ustr struct {
	s string //数据流
	i int    //位置
}

//定义一个字符串创建对象
func NewUstr(s string) *Ustr {
	return &Ustr{s, 0}
}

//读取未读部分的数据长度
func (s *Ustr) Len() int {
	return len(s.s) - s.i
}

func (s *Ustr) Read(p []byte) (n int, err error) {
	for ; s.i < len(s.s) && n < len(p); s.i++ {
		c := s.s[s.i]
		if 'a' <= c && c <= 'z' {
			p[n] = c + 'A' - 'a'
		} else {
			p[n] = c
		}
		n++
	}
	if n == 0 {
		return n, io.EOF
	}
	return n, nil
}

func main() {
	//func ReadFull(r Reader , buf []byte ) (n int,err error)
	//type Reader interface {
	//	Read(p []byte) (n init, err error)
	//}

	s := NewUstr("Hello World! 中国")
	buf := make([]byte, s.Len())
	n, err := io.ReadFull(s, buf)
	fmt.Printf("%s\n", buf) //HELLO WORLD! 中国
	fmt.Println(n, err)     //19 <nil>

	buf1 := bytes.NewBuffer([]byte("Hello World!中国"))
	b1 := make([]byte, buf1.Len())

	//读取所有
	n1, err1 := buf1.Read(b1)
	fmt.Printf("%s %v\n", b1[:n1], err1) //Hello World!中国 <nil>

	buf1.WriteString("Hi Go\n你好 Go")
	buf1.WriteTo(os.Stdout) //Hi Go 换行 你好 Go

	n1, err1 = buf1.Write(b1)
	fmt.Printf("%d %s %v\n", n1, buf1.String(), err1) //18 Hello World!中国 <nil>

	//读取单个
	c, err1 := buf1.ReadByte()
	fmt.Printf("%c %s %v\n", c, buf1.String(), err1) //H ello World!中国

	err = buf1.UnreadByte()
	fmt.Printf("%s %v\n", buf1.String(), err) //Hello World!中国 <nil>

	err = buf1.UnreadByte()
	fmt.Printf("%s %v\n", buf1.UnreadByte(), err)
	//bytes.Buffer: UnreadByte: previous operation was not a read bytes.Buffer: UnreadByte: previous operation was not a read

	io.WriteString(os.Stdout, "Hi Go!\n") //Hi Go!

	r := strings.NewReader("Hi Go!中国")
	b := make([]byte, 6)
	n2, err2 := io.ReadAtLeast(r, b, 3)
	fmt.Printf("%q %d %v\n", b[:n2], n2, err2) //"Hi Go!" 6 <nil>

	r.Seek(0, 0)
	b = make([]byte, 15)
	n, err = io.ReadFull(r, b)
	fmt.Printf("%q %d %v\n", b[:n], n, err) //"Hi Go!中国" 12 unexpected EOF

	//copy
	//cp := make([]byte, 32)
	r1 := strings.NewReader("Hi Go!中国")
	//复制部分
	cn, cerr := io.CopyN(os.Stdout, r1, 7) //Hi Go!�
	fmt.Printf("\n%d %v\n", cn, cerr)      //7 <nil>

	//复制全部
	r1.Seek(0, 0)
	cn, cerr = io.Copy(os.Stdout, r1) //Hi Go!中国
	fmt.Printf("\n%d %v\n", cn, cerr) //12 <nil>

	r1.Seek(0, 0)
	buf2 := make([]byte, 10)
	r2 := strings.NewReader("ABCDEFG")
	cn, cerr = io.CopyBuffer(os.Stdout, r1, buf2) //Hi Go!中国
	fmt.Printf("\n%d %v\n", cn, cerr)             //12 <nil>

	cn, cerr = io.CopyBuffer(os.Stdout, r2, buf2) //ABCDEFG
	fmt.Printf("\n%d %v\n", cn, cerr)             //7 <nil>

	r1.Seek(0, 0)
	lr := io.LimitReader(r1, 5)
	cn, cerr = io.Copy(os.Stdout, lr) //Hi Go
	fmt.Printf("\n%d %v\n", cn, cerr) //5 <nil>

	r3 := strings.NewReader("abc")
	r1.Seek(0, 0)
	r2.Seek(0, 0)
	b = make([]byte, 10)
	mr := io.MultiReader(r1, r2, r3)

	for n, err := 0, error(nil); err == nil; {
		n, err = mr.Read(b)
		fmt.Printf("%q\n", b[:n])
	}
	//"Hi Go!中\xe5"
	//"\x9b\xbd"
	//"ABCDEFG"
	//"abc"
	//""

	r1.Seek(0, 0)
	r2.Seek(0, 0)
	r3.Seek(0, 0)
	mr = io.MultiReader(r1, r2, r3)
	io.Copy(os.Stdout, mr) //Hi Go!中国ABCDEFGabc
	fmt.Println()

	r1.Seek(0, 0)
	mv := io.MultiWriter(os.Stdout, os.Stdout)
	r1.WriteTo(mv) //Hi Go!中国Hi Go!中国
	fmt.Println()

	//创建管道支持读写
	//管道没有缓冲区，读写操作可能会被阻塞。可以安全的对管道进行并行的读、写或关闭
	re, ww := io.Pipe()

	//读取
	go func() {
		buf := make([]byte, 5)
		for n, err := 0, error(nil); err == nil; {
			n, err = re.Read(buf)
			re.CloseWithError(errors.New("关闭了"))
			fmt.Printf("读取：%d,%v,%s\n", n, err, buf)
		}
	}()
	//写
	n, err = ww.Write([]byte("Hi Go!中国"))
	fmt.Printf("写入:%d,%v\n", n, err)
	//读取：5,<nil>,Hi Go
	//读取：0,io: read/write on closed pipe,Hi Go
	//写入:5,关闭了

	rp, wp := io.Pipe()

	go func() {
		buf := make([]byte, 5)
		for n, err := 0, error(nil); err == nil; {
			n, err = rp.Read(buf)
			fmt.Printf("读取：%d,%v,%s\n", n, err, buf[:n])
		}
	}()

	fmt.Println("----\n")
	n, err = wp.Write([]byte("Hi Go!中国"))
	fmt.Printf("写入：%d,%v\n", n, err)

	wp.CloseWithError(errors.New("写关闭"))
	n, err = wp.Write([]byte("Go!"))
	fmt.Printf("写入：%d,%v\n", n, err)
	time.Sleep(time.Second * 1)
	//读取：5,<nil>,Hi Go
	//读取：5,<nil>,!中�
	//读取：2,<nil>,��
	//写入：12,<nil>
	//	写入：0,io: read/write on closed pipe
	//读取：0,写关闭,
}

```

## ioutil：

```go
// Discard 是一个 io.Writer 接口，调用它的 Write 方法将不做任何事情
// 并且始终成功返回。
var Discard io.Writer = devNull(0)

// ReadAll 读取 r 中的所有数据，返回读取的数据和遇到的错误。
// 如果读取成功，则 err 返回 nil，而不是 EOF，因为 ReadAll 定义为读取
// 所有数据，所以不会把 EOF 当做错误处理。
func ReadAll(r io.Reader) ([]byte, error)

// ReadFile 读取文件中的所有数据，返回读取的数据和遇到的错误。
// 如果读取成功，则 err 返回 nil，而不是 EOF
func ReadFile(filename string) ([]byte, error)

// WriteFile 向文件中写入数据，写入前会清空文件。
// 如果文件不存在，则会以指定的权限创建该文件。
// 返回遇到的错误。
func WriteFile(filename string, data []byte, perm os.FileMode) error

// ReadDir 读取指定目录中的所有目录和文件（不包括子目录）。
// 返回读取到的文件信息列表和遇到的错误，列表是经过排序的。
func ReadDir(dirname string) ([]os.FileInfo, error)

// NopCloser 将 r 包装为一个 ReadCloser 类型，但 Close 方法不做任何事情。
func NopCloser(r io.Reader) io.ReadCloser

// TempFile 在 dir 目录中创建一个以 prefix 为前缀的临时文件，并将其以读
// 写模式打开。返回创建的文件对象和遇到的错误。
// 如果 dir 为空，则在默认的临时目录中创建文件（参见 os.TempDir），多次
// 调用会创建不同的临时文件，调用者可以通过 f.Name() 获取文件的完整路径。
// 调用本函数所创建的临时文件，应该由调用者自己删除。
func TempFile(dir, prefix string) (f *os.File, err error)

// TempDir 功能同 TempFile，只不过创建的是目录，返回目录的完整路径。
func TempDir(dir, prefix string) (name string, err error)
```

```go
package main

import (
	"fmt"
	"io/ioutil"
	"os"
)

func main() {
	//读取目录
	rd, err := ioutil.ReadDir("./")
	fmt.Println(err)
	for _, fi := range rd {
		if fi.IsDir() {
			fmt.Printf("[%s]\n", fi.Name())
		} else {
			fmt.Println(fi.Name())
		}
	}

	//临时目录
	dir, err := ioutil.TempDir("", "Test")
	if err != nil {
		fmt.Println(err)
	}
	defer os.Remove(dir) //用完删除
	fmt.Printf("%s\n", dir)

	f, err := ioutil.TempFile(dir, "t.log")
	if err != nil {
		fmt.Println(err)
	}
	defer os.Remove(f.Name())
	fmt.Printf("%s\n", f.Name())
}

```

## bufio：

带缓存的io

```go
package main

import (
	"bufio"
	"fmt"
	"strings"
)

func main() {
	sr := strings.NewReader("ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890")
	//创建带缓存的，size默认16，假如小于16则都为16
	buf := bufio.NewReaderSize(sr, 0)

	//返回缓存中未读取的数据长度
	fmt.Println(buf.Buffered()) //0
	//返回一个切片，该切片用于缓存前n个字节的数据
	//该操作不会将数据读出，只是引用
	s, _ := buf.Peek(5)
	s[0], s[1], s[2] = 'a', 'b', 'c'
	fmt.Printf("%d %q\n", buf.Buffered(), s) //16 "abcDE"

	//跳过后续的n个字节数据
	buf.Discard(1)//跳过a

	b := make([]byte, 10)
	for n, err := 0, error(nil); err == nil; {
		n, err = buf.Read(b)
		fmt.Printf("%d %q %v\n", buf.Buffered(), b[:n], err)
	}
	//5 "bcDEFGHIJK" <nil>
	//	0 "LMNOP" <nil>
	//	6 "QRSTUVWXYZ" <nil>
	//	0 "123456" <nil>
	//	0 "7890" <nil>
	//	0 "" EOF
}

```



```go
package main

import (
	"bufio"
	"fmt"
	"strings"
)

func main() {
	sr := strings.NewReader("ABCD\nEFGHIJKLMNOPQRSTUVWXYZ1234567890")

	buf := bufio.NewReaderSize(sr, 0)

	for line, isPrefix, err := []byte{0}, false, error(nil); len(line) > 0 && err == nil; {
		line, isPrefix, err = buf.ReadLine()
		fmt.Printf("%q %t %v\n", line, isPrefix, err)
	}
	//16
	//"ABCD" false <nil>
	//	"EFGHIJKLMNOPQRST" true <nil>
	//	"UVWXYZ1234567890" true <nil>
	//	"" false EOF

}

```

```go
package main

import (
	"bufio"
	"fmt"
	"strings"
)

func main() {
	sr := strings.NewReader("ABCD\nEFGHIJKLMNOPQRSTUVWXYZ1234567890")

	buf := bufio.NewReaderSize(sr, 0)

	for line, err := []byte{0}, error(nil); len(line) > 0 && err == nil; {
		line, err = buf.ReadSlice('\n')
		fmt.Printf("%q,%v\n", line, err)
	}
	//"ABCD\n",<nil>
	//	"EFGHIJKLMNOPQRST",bufio: buffer full

}

```

### 写入：

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	buf := bufio.NewWriterSize(os.Stdout, 17)

	//17 0  未使用的空间，缓存中未使用的空间
	fmt.Println(buf.Available(), buf.Buffered())

	buf.WriteString("ABCDEF\nGHIJKLMNOPQRSTUVWXYZ\n1234567890")

	fmt.Println(buf.Available(), buf.Buffered())

	//统一输出
	buf.Flush()

	//17 0
	//ABCDEF
	//GHIJKLMNOPQRSTUVWXYZ
	//12345613 4
	//7890%
	//因为超出缓存，所以前34个输出了，
}

```

### Scanner：

```go
package main

import (
	"bufio"
	"fmt"
	"strconv"
	"strings"
)

func main() {
	const input = "1234 5678 1234567901234567890 90"

	scanner := bufio.NewScanner(strings.NewReader(input))

	split := func(data []byte, atEOF bool) (ad int, token []byte, err error) {
		ad, token, err = bufio.ScanWords(data, atEOF)
		if err == nil && token != nil {
			_, err = strconv.ParseInt(string(token), 10, 32)
		}
		return
	}
	//设置匹配函数
	scanner.Split(split)

	//扫描
	for scanner.Scan() {
		fmt.Printf("%s\n", scanner.Text())
	}
	if err := scanner.Err(); err != nil {
		fmt.Printf("错误输入：%s", err)
	}
	//1234
	//5678
	//错误输入：strconv.ParseInt: parsing "1234567901234567890": value out of rang

}

```