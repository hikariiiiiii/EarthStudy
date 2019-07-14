# 语法基础

- Go语言只有值传递，地址（&）是指针（*）的值，指针是地址的变量
- Go语言没有隐式类型转化
- Go语言变量类型写在变量名后面
- Go语言支持封装，不支持继承、多态；只有struct没有class

## var
包内、函数内

``` go
var aa int = 3
var (
	aa int = 3
	ss = ”kkk”
	bb = true
	)
var aa, ss, bb = 3, “kkk”, true
```

函数内

``` go
aa, ss, bb := 3, “kkk”, true
```

## 内建类型
bool, string
(u)int, (u)int8, (u)int16, (u)int32, (u)int64, uintptr（指针）//(u)int根据操作系统32 64决定位数
byte（8位）, rune（32位）
float32, float63, complex64, complex128

## const
``` go
const aa int = 3
const aa, ss, bb = 3, “kkk”, true
const(
	cpp = iota	//iota是自增值
	_
	python
	java
	javascript
)
```


iota在const关键字出现时将被重置为0
const中每新增一行常量声明将使iota计数+1（注意在同一行计数不+1）

## 控制语句
if、for、switch、select
没有while，for即是while
遍历可使用

``` go
for i, v := range xxx {
}
```

## 函数
``` go
func function_name( [parameter list] ) [return_types / return parameter list]
{
	defer xxxx
   	return x	//若有返回参数名则无需return，直接给返回参数赋值
}
```

## Array
数组是值类型，，长度不同类型不同，赋值即拷贝，一般不直接使用数组

``` go
var variable_name [SIZE] variable_type
variable_name = [SIZE] variable_type{v1, v2, v3}
//SIZE为...时根据初始化元素的个数设置大小
```

## Slice
声明：

``` go
var s []Type        //nil 空切片
```

初始化：

``` go
m := []Type{ , , , , }
m := make([]Type, len, cap)	//len长度，cap可达最大长度
m := a[ s:e]	//a可以是已有的slice或数组，前闭后开
				//其中e不能大于数组的长度或slice的cap
slice的len根据可读元素决定，cap根据开辟的内存决定
```

读取元素：

``` go
m[n]				//其中n不能超过len-1
```

遍历：

``` go
for i, v := range s {
} 
```

操作：

``` go
append( s1, v1, v2 )    //若cap不够会重新开辟一个内存，并赋值原值过去
append( s1, s2... )     //...将s2分解为一个个元素
copy(s1, s2)        	//只将覆盖s1 len范围内的值
len(s)					//返回长度
cap(s)					//返回可达最大长度
```

## Map

声明：

``` go
var m map[K]V         //nil
```

初始化：

``` go
m := map[K]V {
    k1 : v1
}
m := make(map[K]V)     //EmptyMap
```

读取：

``` go
Map[K]=v
value , ok := map[K]   //若找不到键值 ok为false
```

遍历：

``` go
for k, v := range m{
}
```

操作：

``` go
delete(map , K)			//删除对应键值
len(m)					//获取元素数量
```

## String、Rune

len(string) 返回的是字节长度，可使用utf8.RuneCountString获得字符数量
range遍历string得到的是pos是字节坐标，不适合处理非unicode编码的字符，可先使用[]rune(string)转换后再遍历

## 结构体
声明：

``` go
type StructName struct{
   var1 int
   var2 *T
}
```

初始化：

``` go
var t StructName
t = StructName {
  1, nil
}
```

使用new函数初始化：

``` go
t = new(StructName)
t.var1 = 1
t.var2 = nil
```

使用工厂创建函数初始化：

``` go
func createStruct(var1 int) *StructName {
	return &StructName { var1, nil }   
	//局部变量的地址也可以返回给别人用，内存分配在堆还是栈由编译器决定
}
```

为结构定义方法：

``` go
// 指针接收者（要改变内容、结构过大、建议有指针接收者则保持一致性也使用指针接收者）
	func (StructName *t) funcname { }
// 值接收者
	func (StructName t) funcname { }
```

**指针能直接调用方法，当做值来用**
**nil指针也可以调用方法**

## 包
一个目录下文件只能有同一个包名
main包有一个可执行入口
结构体的方法必须放在同一个包内，但可在不同文件里

扩展系统类型或别人类型的方法：

- 定义别名，给要扩展的类型创建一个其他名字，再加方法
- 使用组合，将要扩展的类型放进自己的类型中

## Duck Typing
描述事物的外部行为而非内部结构 -- 像鸭子走路，像鸭子叫，像鸭子就是鸭子

- Go严格来说属于结构化类型系统，是类似duck typing（duck typing要动态绑定，Go没有）
- Python 运行时检查，需要注释来说明接口
- C++ 使用模板实现，编译时检查，需要注释来说明接口
- Java 没有duck typing，必须实现接口方法，无需注释

## 接口
接口由使用者定义+组合，实现者无需了解如何组合

接口变量：

- 接口变量包含 {指向的实现者的类型或类型指针，指向的实现者的值或指针}

- 接口变量可指向实现者的值，也可指向实现者的指针

- 接口变量蕴含一个指针，故几乎不需要使用接口的指针

- 若实现的方法是指针接收者，则接口变量必须指向实现者的指针

Type Assertion：将接口变量转换为指定实现者类型

``` go
  v, ok := i.(int)		//接口变量.(实现者类型/类型指针)
```

Type Switch： 判断实现者类型/类型指针

``` go
  switch v := i.(type) {
  	case T1:
  		// TODO
  	case T2:
  		// TODO
  }
```

表示任何类型：

``` go
  v := interface{} 
```

常用接口：

- Stringer接口：相当于tostring
- Reader/Writer接口：文件的抽象

## 嵌入类型
当我们嵌入一个类型，这个类型的方法就变成了外部类型的方法，但是当它被调用时，方法的接受者是内部类型(嵌入类型)，而非外部类型。

## 函数式编程
函数体 = {局部变量，自由变量}
闭包 = 函数体 + 所有相关自由变量指向的值（递归）
斐波那契数列
为函数实现接口

## defer
- 确保调用在函数结束（return或panic）时发生
- 参数在defer语句时计算结果
- 多个defer时，遵循后进先出

常用场景：

Open/Close
Lock/Unlock
PrintHeader/PrintFooter

## error

error接口用于实现自定义错误

``` go
type error interface{
	Error() string
}
```

判断错误类型，MyError为自定义错误类型

``` go
if dError, ok = err.(*MyError); !ok{
	painc("unknown err")
}
else{
	//TODO
}
```

## panic

尽量不要用
## recover
在defer中使用recover()，来保护panic

``` go
defer func(){
	r := recover()
	// TODO
}()
```

## 表格驱动测试
分离测试数据和测试逻辑
更详细的自定义出错信息
可部分失败
Go语言语法更易实现表格驱动测试

## 单元测试testing.T
``` go
func TestTriangle(t *testing.T) {
	tests := []struct{ a, b, c int }{
		{3, 4, 5},
		{5, 12, 13},
		{8, 15, 17},
		{12, 35, 37},
		{30000, 40000, 50000},
	}
	for _, tt := range tests {
		if actual := calcTriangle(tt.a, tt.b); actual != tt.c {
			t.Errorf("calcTriangle(%d, %d); got %d; expected %d",
				tt.a, tt.b, actual, tt.c)
		}
	}
}
```

命令行运行：

``` bash
go test .
```

代码覆盖率：

``` bash
go test -coverprofile=c.out
go tool cover
go tool cover -html=c.out
```


## 性能测试testing.B
``` go
func BenchmarkSubstr(b *testing.B) {
	s := "黑化肥挥发发灰会花飞灰化肥挥发发黑会飞花"
	for i := 0; i < 13; i++ {
		s = s + s
	}
	b.Logf("len(s) = %d", len(s))
	ans := 8
	b.ResetTimer()  //排除准备数据的时间

	for i := 0; i < b.N; i++ {
		actual := lengthOfNonRepeatingSubStr(s)
		if actual != ans {
			b.Errorf("got %d for input %s; "+
				"expected %d",
				actual, s, ans)
		}
	}
}
```
命令行运行 

``` bash
go test -bench .
```

性能分析：

``` bash
go test -bench . -cpuprofile cpu.out
go tool pprof cpu.out
help
web
quit
```

## http测试：

**1. 通过使用假的Request/Response**

   ``` go
   response := httptest.NewRecorder()
   request := httptest.NewRequest( http.MethodGet, "http://xxxxxxxxxx", nil)
   ```

**2. 通过起服务器**

``` go
   server := httptest.NewServer( http.HandlerFunc(f))
   resp, _ := http.Get(server.URL)
```

## 文档
命令行运行 

``` bash
go doc
```

命令行指令帮助 

``` bash
go help doc
```

启动帮助文档服务器 

``` bash
godoc -http :6060
```

## Example
Example是另一种测试，也可运行

在给文档提供示例

	func ExampleTypename_Funcname() {
		// 函数调用，期望的函数返回结果
	}
## goroutine
协程Coroutine：

- 轻量级“线程”

- 非抢占式多任务处理，由协程主动交出控制权

- 编译器/解释器/虚拟机层面的多任务，GO语言有自己的调度器

- 多个协程可能在一个或多个线程上运行

- 子程序是协程的一个特例

**C++：**Boost.Coroutine

**Java：**不支持

**Python：**使用yield关键字、async def

**Go语言：**goroutine

``` go
  go func() {
  	// TODO
  	runtime.Gosched();  //让出控制权
  }()
```

- 任何加上go就能送给调度器运行

- 不需要在定义时区分是否是异步函数

- 调度器会在合适的点进行切换

- 切换点：

  1. I/O，select

  2. channel

  3. 等待锁

  4. 函数调用（有时）

  5. Runtime.Gosched()

     以上只是参考，不能保证切换，不能保证在其他地方不切换

- 使用-race来检测数据访问冲突 

  ``` bash
  go run -race xxxx.go
  ```

## channel
类似Python的yield

当一个channel的数据未传输成功时（未发、未收、无缓冲区），当前协程会阻塞等待

声明：

``` go
var c chan int			//可发可收
var c chan<- int		//send only type 只能发送数据的channel
var c <-chan int		//只能收取数据
```

初始化：

``` go
c := make(chan int)	
c := make(chan int, 3)	//创建一个缓冲区大小为3的channel
```

操作：

``` go
c<-1				    //发送数据
n := <-c		 		//收取数据
close(c)				//关闭channel，之后再收取的数据都为数据类型的初始值 
n, ok := <-c			//使用ok判断是否发完数据
for n := range c {
	//收取直到channel关闭
}
```

## Go语言channel基于CSP模型

“不要通过共享内存来通信；通过通信来共享内存”，可创建两个channel来完成双向发送数据

## WaitGroup
利用WaitGroup判断协程工作是否完毕

``` go
var wg = sync.WaitGroup
wg.add(1)	    //添加任务数
wg.done()	    //完成一个任务
wg.wait()		//挂起等待所有任务完成
```

## select
利用select来进行调度，实现非阻塞式获取channel数据，select中若case不可运行且没有default则阻塞直到有case可运行

``` go
select {
    case n:= <- c1
    case n:= <- c2
    default 
}
```

注：在select中使用Nil Channel时会直接跳过而非阻塞

时间channel：

``` go
time.After(10 * time.Second)	//倒计时
time.Tick(time.Second)			//周期定时触发
```

## 传统同步机制
非CPS模型，在GO中尽量少使用
WaitGroup
MuteX
Cond

## http标准库
``` go
http.Get(“https://xxxxxxxxxxxxxx.com”)
request, err := http.NewRequest(http.MethodGet,
		"https://xxxxxxxxxxxxxxx.com", nil)
request.Header.Add("","")
client := http.DefaultClient()
client := http.Client{
	xxx : xxx
	CheckRedirect : func(){
    },
}
client.Do(request)
httputil.DumpResponse(resp, true)
```

http调试：

1. 浏览器访问 http服务地址/debug/pprof
2. 命令行go tool pprof 服务地址/debug/pprof/profile获得30秒服务的CPU使用情况
   结束后对话输入web
3. 命令行go tool pprof 服务地址/debug/pprof/heap 获得服务的内存使用情况
   结束后对话输入web

## 其他标准库
bufio
log
encoding/json
regexp
time
string/math/rand

使用 godoc -http:6060 启动帮助文档服务器



