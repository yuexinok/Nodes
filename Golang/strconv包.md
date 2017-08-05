# strconv包

```go
package main

import (
	"fmt"
	"strconv"
	"unicode/utf8"
)

func main(){
	//bool转字符串
	fmt.Println(strconv.FormatBool(true)) //字符串true

	//转字符串为bool
	//"1", "t", "T", "true", "TRUE", "True":
	//"0", "f", "F", "false", "FALSE", "False"
	b1,_ := strconv.ParseBool("1")
	fmt.Println(b1) //true

	//整型转字符串 第一个参数为具体的值，第2个表示进制
	fmt.Println(strconv.FormatInt(-322,10)) //-322
	fmt.Println(strconv.FormatUint(322,10)) //322

	//字符串转整型
	i1,_ := strconv.ParseInt("-213a21",10,0)
	fmt.Println(i1)//0
	i2,_ := strconv.ParseInt("-213",10,0)
	fmt.Println(i2)//-213
	i3,_ := strconv.ParseUint("-213",10,0)
	fmt.Println(i3) //0

	//10进制快速处理 int->string
	fmt.Println(strconv.Itoa(-213))//-213
	//返回两个值
	fmt.Println(strconv.Atoi("-213"))//-213 <nil>

	//float
	// fmt：格式标记（b、e、E、f、g、G）
	// prec：精度（数字部分的长度，不包括指数部分）
	// bitSize：指定浮点类型（32:float32、64:float64），结果会据此进行舍入。
	fmt.Println(strconv.FormatFloat(12.32,'f',1,64))//12.3
	fmt.Println(strconv.ParseFloat("-1222.321",32))//-1222.321044921875 <nil>

	//// 字符串中不能含有控制字符（除了 \t）和“反引号”字符，否则返回 false
	for i := rune(0);i< utf8.MaxRune;i++ {
		if !strconv.CanBackquote(string(i)) {
			fmt.Printf("%q,",i)
		}
	}
	//'\x00','\x01','\x02','\x03','\x04','\x05','\x06','\a','\b','\n','\v','\f','\r','\x0e','\x0f','\x10','\x11','\x12','\x13','\x14','\x15','\x16','\x17','\x18','\x19','\x1a','\x1b','\x1c','\x1d','\x1e','\x1f','`','\u007f','\ufeff',%
	fmt.Println(strconv.IsPrint(' '))//true
	fmt.Println(strconv.IsPrint('\t'))//false

	//IsGraphic 是否为图形字符

	//将特殊字符格式
	s := "'He'llo \t 世 界 \"!\n哈\r哈"
	fmt.Println(s)//换行展示了
	fmt.Println(strconv.Quote(s))//"'He'llo \t 世 界 \"!\n哈\r哈"
	fmt.Println(strconv.QuoteToASCII(s))//"'He'llo \t \u4e16 \u754c \"!\n\u54c8\r\u54c8"
	fmt.Println(strconv.QuoteToGraphic(s)) //"'He'llo \t 世 界 \"!\n哈\r哈"

	s1 := `'He'llo \t 世 界 \"!\n哈\r哈`
	fmt.Println(s1) //'He'llo \t 世 界 \"!\n哈\r哈
	fmt.Println(strconv.Quote(s1)) //"'He'llo \\t 世 界 \\\"!\\n哈\\r哈"

	fmt.Println(strconv.Unquote(strconv.Quote(s1)))//'He'llo \t 世 界 \"!\n哈\r哈 <nil>

	s3 := []byte{'A','B'}
	fmt.Println(strconv.AppendInt(s3,0,10))//[65 66 48]

}

```

## 特别提醒：

1，`` 包裹其实已经被quote的了。

2，byte的范围，是单个的字符，不包含中文的。