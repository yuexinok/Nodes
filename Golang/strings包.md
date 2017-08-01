# strings包使用

```go
	//大小写转化
	fmt.Println(strings.ToUpper("xxssfd")) //XXSSFD
	fmt.Println(strings.ToLower("sEdas"))  //sedas
	fmt.Println(strings.ToTitle("婚纱hellow大Hi man"))

	//清理
	fmt.Println(strings.Trim("我我是啥撒谎说", "我说"))      //是啥撒谎
	fmt.Println(strings.TrimLeft("我我是啥撒谎说", "我说"))  //是啥撒谎说
	fmt.Println(strings.TrimRight("我我是啥撒谎说", "我说")) //我我是啥撒谎
	fmt.Println([]rune("测试啊"))                      //[27979 35797 21834]
	n := 0
	fmt.Println(strings.TrimFunc("我我头是头电话撒头", func(s rune) bool { //是头电话撒
		n++
		c := string(s)    //传入的是第一个字符和第最后一个字符
		fmt.Println(c, n) // 我 1 我 2 头 3 是 4 头 5 撒 6
		if c == "我" {
			return true
		}
		if c == "头" {
			return true
		}
		return false
	}))

	//去除空格
	fmt.Println(strings.TrimSpace(" 撒时候 附带 ds "))       //撒时候 附带 ds
	fmt.Println(strings.TrimPrefix("我的时间啊的.doc", "我啊")) //我的时间啊的.doc
	//严格匹配的
	fmt.Println(strings.TrimPrefix("我的时间啊的.doc", "我的")) //时间啊的.doc
	fmt.Println(strings.TrimSuffix("我的.doC", "doc"))    //我的.doC
	fmt.Println(strings.TrimSuffix("我的.doc", "doc"))    //我的.

	//分隔
	v := strings.Split("你|我|他", "|")
	fmt.Println(v, len(v))             //[你 我 他] 3
	v1 := strings.Split("你 我 他", "  ") //双空格
	fmt.Println(v1, len(v1))           //[你 我 他] 1 没有匹配

	v2 := strings.SplitN("你|我|他", "|", 2)
	fmt.Println(v2, len(v2)) //[你 我|他] 2

	//分割 但是保持分割符号
	v3 := strings.SplitAfter("你|我|他", "|")
	fmt.Println(v3, len(v3)) //[你| 我| 他] 3

	v4 := strings.Fields("你 我  他")
	fmt.Println(v4, len(v4)) //[你 我 他] 3

	//连接
	arr1 := []string{"你","我","他"}
	v5 := strings.Join(arr1,"#")
	fmt.Println(v5)//你#我#他

	//重复
	v6 := strings.Repeat("你我",3)
	fmt.Println(v6)//你我你我你我


	//查找
	fmt.Println(strings.HasPrefix("你我他","你")) //true
	fmt.Println(strings.HasPrefix("你我他","你我")) //true
	fmt.Println(strings.HasPrefix("你我他","我")) //false
	fmt.Println(strings.HasSuffix("你我.jpg","jPg"))//false
	fmt.Println(strings.HasSuffix("你我.jpg",".jpg"))//true

	//是否包含
	fmt.Println(strings.Contains("你我他啊","我啊"))//false
	fmt.Println(strings.Contains("你我他啊","我他"))//true
	fmt.Println(strings.ContainsRune("你我他啊",'我'))//true
	fmt.Println(strings.ContainsAny("你我他啊","我啊"))//true

	//查找索引
	fmt.Println(strings.Index("你我他啊","你"))//0
	fmt.Println(strings.Index("你我他啊","我啊"))//-1
	fmt.Println(strings.IndexAny("你我他啊","我啊"))//3 一个汉字3位 2*3=6
	//fmt.Println(strings.IndexByte("你我他啊",'我')) 报错 因为constant 25105 overflows byte
	fmt.Println(strings.IndexFunc("你我他啊",func(s rune ) bool{
		fmt.Println(string(s)) //20320你 25105我 20182他 一个汉字3位 2*3=6
		if s == '他' {//这里用单引号 因为是rune类型
			return true
		}
		return false
	})) //6

	fmt.Println(strings.LastIndex("你我你他啊他ok我","他"))//15 3*5
	fmt.Println(strings.LastIndex("你我你他啊他ok我","我"))//20 15 + 2*2
	fmt.Println(strings.LastIndex("你我你他啊他ok我","o"))//到达他的时候是15 然后过他 则为1
	fmt.Println(strings.LastIndex("你我你他啊他ok我","k"))//19
	//LastIndexAny
	//LastIndexFunc

	//计数
	fmt.Println(strings.Count("你我他你ok他","他"))//2

	//替换
	fmt.Println(strings.Replace("你我他你啊我啊","他啊","#",2))//你我他你啊我啊
	fmt.Println(strings.Replace("你我他你啊我啊","我他","#",2))//你#你啊我啊
	fmt.Println(strings.Map(func(m rune) rune {
		if m == '你' {
			return '%'
		}
		return m
	},"你我他你啊我啊"))//%我他%啊我啊

	reader := strings.NewReader(`
	我的
	你的 爱谁谁
	wjhhh搜索
	`)
	fmt.Println(n)
	fmt.Println(reader.Len())//41
	fmt.Println(reader.Size())//41
	reader.Reset("你我")
	fmt.Println(reader.Len())//6
	fmt.Println(len("你我"))//6

	re := strings.NewReplacer("你我他你啊我啊")
	re.Replace()


}

```

