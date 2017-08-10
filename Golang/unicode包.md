# Unicode:

## utf8包

```go
package main

import (
	"fmt"
	"unicode/utf8"
)

func main() {
	b := make([]byte, utf8.UTFMax)
	n := utf8.EncodeRune(b, '国')
	fmt.Printf("%v:%v\n", b, n) //[229 155 189 0]:3

	r, n := utf8.DecodeRune(b)
	fmt.Printf("%c:%v\n", r, n) //国:3

	s := "大家好"
	for i := 0; i < len(s); {
		fmt.Println(s[i:]) //大家好 家好 好
		r, n = utf8.DecodeRuneInString(s[i:])
		fmt.Printf("%c:%v \n", r, n) //大:3 家:3
		i += n
	}
	fmt.Println()

	for i := len(s); i > 0; {
		r, n = utf8.DecodeLastRuneInString(s[:i])
		fmt.Printf("%c:%v", r, n) //好:3家:3大:3
		i -= n
	}
	fmt.Println()
	//b = []byte('好') //错误
	b = []byte("好") //[229 165 189]
	fmt.Println(b)

	fmt.Printf("%t,", utf8.FullRune(b))
	fmt.Printf("%t,", utf8.FullRune(b[1:]))
	fmt.Printf("%t,", utf8.FullRune(b[2:]))
	fmt.Printf("%t,", utf8.FullRune(b[:1]))
	fmt.Printf("%t,", utf8.FullRune(b[:2]))
	fmt.Println() //true,true,true,false,false,

	b = []byte("大家好")
	fmt.Println(utf8.RuneCount(b)) //3
	fmt.Println(len(b))            //9

	fmt.Printf("%d,", utf8.RuneLen('A'))
	fmt.Printf("%d,", utf8.RuneLen('\u03A6'))
	fmt.Printf("%d,", utf8.RuneLen('好'))
	fmt.Printf("%d", utf8.RuneLen('\U0010FFFF'))
	//1,2,3,4


	fmt.Println()

	//是否是rune的第一个
	fmt.Printf("%t,", utf8.RuneStart("好"[0])) //true
	fmt.Printf("%t,", utf8.RuneStart("好"[1])) //false
	fmt.Println()

	b = []byte("你好")
	fmt.Printf("%t,",utf8.Valid(b))//true
	fmt.Printf("%t,",utf8.Valid(b[1:]))//false
	fmt.Printf("%t,",utf8.Valid(b[3:]))//true 3位好

	fmt.Println()

	fmt.Printf("%t,",utf8.ValidRune('好')) //true
	fmt.Printf("%t,",utf8.ValidRune(0)) //true
	fmt.Printf("%t,",utf8.ValidRune(0xD800))//false代理区字符
	fmt.Printf("%t,",utf8.ValidRune(0x10FFFFFF)) //false  超出范围

}
```

## rune相关：

```go
package main

import (
	"fmt"
	"unicode"
)

func main() {

	//主要是针对rune类型

	//判断汉字
	for _, r := range "Hello 世界！" {
		if unicode.Is(unicode.Scripts["Han"], r) {
			fmt.Printf("%c", r) //世界
		}
	}
	fmt.Println()
	//判断大小写
	fmt.Println(unicode.IsUpper('H'))        //true
	fmt.Println(unicode.ToUpper('h'))        //72
	fmt.Printf("%c\n", unicode.ToUpper('h')) //H
	fmt.Println(unicode.IsLower('h'))        //true
	//是否为标题
	fmt.Println(unicode.IsTitle('h')) //false
	fmt.Println(unicode.IsTitle('H')) //false
	fmt.Println(unicode.IsTitle('ᾏ')) //true
	fmt.Println(unicode.IsTitle('好')) //false

	fmt.Printf("%c\n", unicode.ToTitle('h'))               //H
	fmt.Printf("%c\n", unicode.To(unicode.LowerCase, 'H')) //h
	fmt.Printf("%c\n", unicode.To(unicode.TitleCase, 'h')) //H

	fmt.Printf("%c\n", unicode.SpecialCase(unicode.CaseRanges).ToUpper('H')) //H

	s := "ΦφϕkKK"
	for _, c := range s {
		//3a6 3c6 3d5 6b 4b 212a
		fmt.Printf("%x ", c)
	}
	fmt.Println()

	for _, c := range s {
		fmt.Printf("%v,%c->%v,%c\n", c, c, unicode.SimpleFold(c), unicode.SimpleFold(c))
	}
	//934,Φ->966,φ
	//966,φ->981,ϕ
	//981,ϕ->934,Φ
	//107,k->8490,K
	//75,K->107,k
	//8490,K->75,K
	fmt.Println()
	//是否是10进制数字
	for _, r := range "Hello 123１２３一二三！" {
		if unicode.IsDigit(r) {
			fmt.Printf("%c,", r) //1,2,3,１,２,３,
		}
	}
	fmt.Println()

	//是否是数字
	for _, r := range "Hello 123１２３一二三！" {
		if unicode.IsNumber(r) {
			fmt.Printf("%c,", r) //1,2,3,１,２,３,
		}
	}
	fmt.Println()

	//字母
	for _, r := range "Hello\n\t世界！" {
		if unicode.IsLetter(r) {
			fmt.Printf("%c,", r) //H,e,l,l,o,世,界,
		}
	}
	fmt.Println()

	//空白字符
	for _, r := range "Hello \t世　界！\n" {
		if unicode.IsSpace(r) {
			fmt.Printf("%q,", r) //' ','\t','\u3000','\n',
		}
	}
	fmt.Println()
	//控制符号
	for _, r := range "Hello\n\t世界！" {
		if unicode.IsControl(r) {
			fmt.Printf("%#q,", r) //'\n','\t',
		}
	}
	fmt.Println()
	//可打印
	for _, r := range "Hell\no　世界! \t " {
		if unicode.IsPrint(r) {
			fmt.Printf("%c,", r) //H,e,l,l,o,世,界,!, , ,
		}
	}
	fmt.Println()
	//图形
	for _, r := range "Hel\nlo　世界！\t" {
		if unicode.IsGraphic(r) {
			fmt.Printf("%c,", r) //H,e,l,l,o,　,世,界,！,
		}
	}
	fmt.Println()
	//掩码
	for _, r := range "Hello ៉៊់៌៍！" {
		if unicode.IsMark(r) {
			fmt.Printf("%c,", r) // ៉,៊,់,៌,៍,
		}
	}
	fmt.Println()

	//标点
	for _, r := range "He-ll,o 世。界！" {
		if unicode.IsPunct(r) {
			fmt.Printf("%c,", r) //-,,,。,！,
		}
	}
	fmt.Println()

	//符号
	for _, r := range "Hell。o (<世=界>)" {
		if unicode.IsSymbol(r) {
			fmt.Printf("%c,", r) //<,=,>,
		}
	}
	fmt.Println()

	//判断汉字和标点符号
	set := []*unicode.RangeTable{unicode.Han, unicode.P}
	for _, r := range "Hell.o 世界！" {
		if unicode.IsOneOf(set, r) {
			fmt.Printf("%c,", r) //.,世,界,！,
		}
	}
	fmt.Println()

	//输出所有mark字符
	for _, cr := range unicode.M.R16 {
		Lo, Hi, Stride := rune(cr.Lo), rune(cr.Hi), rune(cr.Stride)

		for i := Lo; i >= Lo && i <= Hi; i += Stride {
			if unicode.IsMark(i) {
				fmt.Printf("%c", i)
			}
		}
		// ̴̵̶̷̸ࣰࣱࣲ̡ࣣࣦࣩ࣭࣮࣯ࣶࣹࣺ̀́̂̃̄̅̆̇̈̉̊̋̌ࣔࣕࣖࣗࣘࣙࣚࣛࣜࣝࣞࣟ࣠࣡ࣤࣥࣧࣨ࣪࣫࣬ᢅᢆᢩᤠᤡᤢᤣᤤᤥᤦᤧᤨᤩᤪᤫᤰᤱᤲᤳᤴᤵ⃒⃓⃘⃙⃚᷽᷿᷻᷾⃐⃑⃔⃕⃖⃗⃛⃜᷼⃝⃞ꣅ꣠꣡꣢꣣꣤꣥꣦꣧꣨꣩꣪꣫꣬꣭꣮꣯꣰꣱ꤦ%
	}

}

```

