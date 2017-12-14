# http使用

## GET请求

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	res, err := http.Get("http://baidu.com")
	if err != nil {
		fmt.Println(err)
	}
	defer res.Body.Close()

	body, _ := ioutil.ReadAll(res.Body)
	fmt.Println(string(body))

}
```

```go
package main

import (
	"net/http"
	"log"
	"io/ioutil"
	"fmt"
)

func main() {
	client := &http.Client{}
	resp,err := client.Get("http://baidu.com")
	if err != nil {
		log.Fatalln(err)
	}

	body,err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatalln(err)
	}

	if err := resp.Body.Close();err != nil {
		log.Fatalln(err)
	}
	fmt.Println(string(body))
}

```



## POST json

```go
package main

import (
	"bytes"
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	body := `
{
	"code":200,
	"name":"哈哈"
}`
	res, err := http.Post("http://bigbig.me", "application/json;charset=utf-8", bytes.NewBuffer([]byte(body)))
	if err != nil {
		fmt.Println("Fatal error", err.Error())
	}

	defer res.Body.Close()

	if res.StatusCode == http.StatusOK {
		fmt.Println("请求成功！")
	}

	content, err := ioutil.ReadAll(res.Body)

	if err != nil {
		fmt.Println("Fatal error:", err.Error())
	}
	fmt.Println(string(content))

}
```

## POST表单提交

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"net/url"
	"strings"
)

func main() {
	v := url.Values{}
	v.Set("name", "borgxiao")
	v.Set("age", "21")

	//URL编码
	body := ioutil.NopCloser(strings.NewReader(v.Encode()))
	client := &http.Client{}

	request, err := http.NewRequest("POST", "http://www.bigbig.me", body)

	if err != nil {
		fmt.Println("Fatal error", err.Error())
	}
	request.Header.Set("Content-Type", "application/x-www-form-urlencoded;param=value")
	resp, err := client.Do(request)
	defer resp.Body.Close()
	content, err := ioutil.ReadAll(resp.Body)

	if err != nil {
		fmt.Println("Fatal error", err.Error())
	}

	fmt.Println(string(content))
}

```

```go
package main

import (
	"net/http"
	"net/url"
	"log"
	"io/ioutil"
	"fmt"
)

func main() {
	client := &http.Client{}
	data := url.Values{}
	data.Add("name","bnehd")
	resp,err := client.PostForm("http://127.0.0.1:9000",data)
	if err != nil {
		log.Fatalln(err)
	}
	body,err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatalln(err)
	}
	fmt.Println(string(body))
}

```



## WEB服务

```Go
package main

import (
    "net/http"
)

func SayHello(w http.ResponseWriter, req *http.Request) {
    w.Write([]byte("Hello Go!"))
}

func main() {
  	//注册handler
    http.HandleFunc("/hi", SayHello)
  	//启动服务
    http.ListenAndServe(":80", nil)

}
```

```go
package main

import (
	"net/http"
	"fmt"
	"log"
)

const html = `<form method="post">
<input type="text" name="name">
<input type="submit" value="Send">
</form>`

func GetFormHandler(w http.ResponseWriter,r *http.Request) {
	w.Header().Set("Content-Type","text/html")
	if r.Method == "POST" {
		name := r.FormValue("name")
		if name != "" {
			fmt.Fprintln(w,"Received",name)
			return
		}
	}
	fmt.Fprintln(w,html)
}
func main()  {
	http.HandleFunc("/",GetFormHandler)
	log.Fatal(http.ListenAndServe(":9000",nil))
}

```

```go
package main

import (
	"net"
	"net/http"
	"fmt"
	"time"
	"log"
)

func PrintState(conn net.Conn,state http.ConnState) {
	fmt.Println(state)
}

func homeHandler(w http.ResponseWriter,r *http.Request) {
	fmt.Fprintln(w,"<h1>Homepage</h1>")
}

func main() {
	mux := http.NewServeMux()

	mux.HandleFunc("/",homeHandler)
	server := &http.Server{
		Addr:":9000",
		Handler:mux,
		ReadTimeout:10*time.Second,
		WriteTimeout:10*time.Second,
		ConnState:PrintState,
	}

	go server.ListenAndServe()

	ln,err := net.Listen("tcp",":9001")
	if err != nil {
		log.Fatalln(err)
	}

	log.Fatal(server.Serve(ln))
}

```



## web文件服务系统

```go
package main

import (
	"log"
	"net/http"
)

func main() {
	log.Fatal(http.ListenAndServe(":9000",http.FileServer(http.Dir("."))))
}

```



## CooKie读写操作

```Go
package main

import (
	"net/http"
	"time"
	"fmt"
	"log"
)

func HandlerOne(w http.ResponseWriter,r *http.Request) {
	cookie := &http.Cookie{Name:"xiaoxiao",Value:"ddddd",Expires:time.Now().Add(time.Hour * 24)}

	http.SetCookie(w,cookie)

	fmt.Fprintln(w,"set cookie")

}

func HandlerTwo(w http.ResponseWriter,r *http.Request) {
	cookie,err := r.Cookie("xiaoxiao")
	if err != nil {
		http.Error(w,err.Error(),http.StatusInternalServerError)
		return
	}

	fmt.Fprintln(w,"cookie read",cookie.Value)
}

func main() {
	http.HandleFunc("/",HandlerOne)
	http.HandleFunc("/two",HandlerTwo)
	log.Fatal(http.ListenAndServe(":9000",nil))
}

```

