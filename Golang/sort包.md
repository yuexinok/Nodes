# sort包：

满足Interface接口类型的都可以本sort的函数排序

```go
type Interface interface {
  Len() int
  Less(i,j int) bool //报告索引i的元素是否比索引j的元素小
  Swap(i,j int) //交换索引i和j的两个元素位置
}
//默认升序
func Sort(data Interface)
func Stable(data Interface)
//降序，不改变元素顺序，只改变排序行为
func Reverse(data Interface) Interface
//判断是否已经排序
func IsSorted(data Interface) bool
```

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	i := []int{3, 7, 1, 3, 6, 9, 4, 1, 8, 5, 2, 11, 0}
	a := sort.IntSlice(i)
	fmt.Println(sort.IsSorted(a)) //false
	sort.Sort(a)
	fmt.Println(a)                //[0 1 1 2 3 3 4 5 6 7 8 9 11]
	fmt.Println(sort.IsSorted(a)) //true

	b := sort.IntSlice{3}
	fmt.Println(sort.IsSorted(b)) //true
	fmt.Println(b)                //[3]

	//更改排序行为
	c := sort.Reverse(a)
	fmt.Println(sort.IsSorted(c)) //false
	fmt.Println(c)                //&{[0 1 1 2 3 3 4 5 6 7 8 9 11]}
	sort.Sort(c)
	fmt.Println(c)                //&{[11 9 8 7 6 5 4 3 3 2 1 1 0]}
	fmt.Println(sort.IsSorted(c)) //true

	//再次更改排序行为
	d := sort.Reverse(c)
	fmt.Println(sort.IsSorted(d)) //false
	fmt.Println(d)                //&{0xc42000e300}
	sort.Sort(d)
	fmt.Println(d)                //&{0xc42000e300}
	fmt.Println(sort.IsSorted(d)) //true

}

```

## 不同类型的排序：

### 整数：

```go
//对a进行升序排列
func Ints(a []int)
//是否为升序排列
func IntsAreSorted(a []int) bool
//搜索a中值为x的索引，如果找不到，则返回最接近且大于x的值的索引
func SearchInts(a []int,x int) int

```

```Go
package main

import (
	"fmt"
	"sort"
)

func main() {
	i := []int{3, 7, 1, 3, 6, 9, 4, 1, 8, 5, 2, 11, 0}
	sort.Ints(i)
	fmt.Println(i)                     //[0 1 1 2 3 3 4 5 6 7 8 9 11]
	fmt.Println(sort.IntsAreSorted(i)) //true
	a := sort.SearchInts(i, 10)
	fmt.Println(i[a]) //11

}

```

```go
type IntSlice []int
func (p IntSlice) Len() int
func (p IntSlice) Less(i,j int) bool
func (p IntSlice) Swap(i,j int)
func (p IntSlice) Sort()
func (p IntSlice) Search(x int) int
```



```Go
i := []int{3, 7, 1, 3, 6, 9, 4, 1, 8, 5, 2, 11, 0}
a := sort.IntSlice(i) //强制转换
a.Sort()
fmt.Println(a)                //[0 1 1 2 3 3 4 5 6 7 8 9 11]
fmt.Println(sort.IsSorted(a)) //true
i1 := a.Search(6)
fmt.Println(i1, a[i1]) //8 6
```

### float：

```go
func Float64s(a []float64)
func FloatsAreSorted(a []float64) bool
func SearchFloat64s(a []float64,x float64) int

type Float64Slice []float64
func (p Float64Slice) Len() int
func (p Float64Slice) Less(i,j int) bool
func (p Float64Slice) Swap(i,j int)
func (p Float64Slice) Search(x float64) int
```

### Strings：

```go
func Strings(a []string)
func StringsAreSorted(a []string )bool
func SearchStrings(a[]string,x string) int

type StringSlice []string
func (p StringSlice) Len() int
func (p StringSlice) Less(i,j int) bool
func (p StringSlice) Swap(i,j int)
func (p StringSlice) Sort()
func (p StringSlice) Search(x string) int
```

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	a := sort.StringSlice{"Hello", "Wolrd", "Golang", "Sort", "Nice"}
	a.Sort()
	fmt.Println(a)
	i := sort.Search(len(a), func(i int) bool {
		return len(a[i]) > 0 && a[i][0] > 'N'
	})
	fmt.Println(a[i])
	//[Golang Hello Nice Sort Wolrd]
	//Sort
}

```