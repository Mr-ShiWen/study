# 包管理

java最终的结构是本项目结构，import路径也是从其他jar包的自身项目路径开始的。项目本身不依赖外部路径
go的最终结构是项目的网络路径（module path=项目的网络path），依赖import的也是其他项目的网络路径，下载的依赖也保留网络路径结构，可以认为网络路径是项目本身属性


不同于java包就是目录，go里面的包不等同于目录，如果一个包在某目录下，可以认为这个包是附属在这个目录上的，如果一个包不再任何目录下，如src下的包，则可以认为这个包不附属在某个目录上。也就是说，包和目录是独立的，它们之间如果存在关系的话就是附属关系。

​		1、一个目录最多一个包。

​        2、import时，需要import包的全路径。import "a/b/c"，指import目录a/b/c下的包，这个包名不一定是c，使用时需要  包名.方法 的方式使用。

```go
package main

import (
	"fmt"
	"project1/pkg1"
)


func main() {
	fmt.Println("I am project1");
	pkg2.Yell();//这里的pkg2是 project1/pkg1 这个目录下的包名
}
```

​		当不同的xx.go源文件属于同一个包时，它们之间其实会被合并为一个源文件，import时是import某个包，调用时是   xx包.YY()   这样调用方法。



关于包的管理，也就是说import时到底从那么寻找我们所需要的包，主要有三种模式：go path 、go vender 、go mod。 

具体见：https://www.cnblogs.com/wongbingming/p/12941021.html。



## go path模式

* step1：设置环境变量 GO111MODULE为off 或者auto（使得import通过 GOPATH 来加载 ），具体在cmd上执行：

  ```txt
  go env -w GO111MODULE=off
  ```

  或者  

  ```
  go env -w GO111MODULE=auto
  ```
  
  
  
* step2：goland里设置`Project GOPATH`为当前项目，在当前项目下创建bin、pkg、src这三个文件夹。

完成上述步骤后，按照规定import包即可。

**特点**：

* 本项目和go get拉取的以来都放在 GOPATH/src 下，混在一起
* 不同项目最好用不同GOPATH，以便区分。但这样又导致不同项目的依赖不能共用，需要各自拉取







## go vendor模式

go vendor模式，是基于go path模式的，只是在相应包旁边建立vendor文件夹，该包import时，优先从vendor里面寻找，再到GOPATH/src下面寻找。其优先顺序见：

https://www.cnblogs.com/wongbingming/p/12941021.html





## go mod 模式

* step1：设置环境变量 GO111MODULE为on 或者auto（使得import通过 GOPATH 来加载 ），具体在cmd上执行：

  ```txt
  go env -w GO111MODULE=on
  ```

  或者

  ```txt
  go env -w GO111MODULE=auto
  ```

* step2：创建go mod项目即可。或者在项目目录下执行 go mod init <我的项目>，[具体见这里](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483713&idx=1&sn=817ffef56f8bc5ca09a325c9744e00c7&source=41#wechat_redirect)



一个项目可以看做是一个module，里面有两个核心文件go.mod与go.sum。

**特点**：

* 自己项目随便放，不局限于GOPATH。通过go.mod文件指明引用的依赖
* 拉取的依赖放在GOPATH/pkg/mod下
* 不同项目依赖可以共用，因为依赖都是在GOPATH/pkg/mod下



**注意**：go.mod 的 module格式为：< url>/projectDir , 否则自己跑没问题，但是不能被引用，因为引用时会按照< url>/projectDir下载，但是检查发现module不是这个的话报错



参考：https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483713&idx=1&sn=817ffef56f8bc5ca09a325c9744e00c7&source=41#wechat_redirect。









## 总结

1、包/模块路径

每个包或者模块都对应着一个路径，包或者模块在本地或者网络上保存时，需要按照其路径来保存的。本地的模块由于在go.mod里面指定了路径，因此可以不用按照路径来保存。

​		go path模式下：所有的包（自己的包和第三方的包）都放在 GOPATH/src 下。

​		go mod模式：自己包放在自己的module里，第三包放在 GOPATH/pkg/mod 里面。



2、包管理

所谓包管理，核心是到哪里去寻找包。

go path模式下，所有依赖的包都只能去 `GOPATH/src` 目录下寻找，用go get下载的包也保存在 `GOPATH/src` 目录下。

go mod模式下，依赖的包可以在本module下寻找，找不到再到 `GOPATH/pkg/mod` 下寻找。用go get下载的包也保存在`GOPATH/pkg/mod`目录下。



3、一个module依赖的包会加载`GOPATH/pkg/mod` 下，实际上会加载包含该包的整个module，如果这个module还有依赖，那就继续加载依赖，直到所有依赖加载完毕。



4、github模块（即可以从github下载到的模块），其路径形式为[ github.com/xx/yy ]，xx对应github用户名，yy对应相应的 repository。





# 相关命令

* **go install xxx** ：就是编译xxx，**xxx是一个包含main包的目录，可以是多级目录，只要最终目录包含main包即可**，实际上是GOPATH/src/xxx，与当前所在位置无关。编译后在GOPATH/bin目录下得到编译后的二进制目标执行文件，该文件命名是以最终目录命名的。

* **go build xxx.go** ：  编译xxx.go文件，生成可执行的目标文件，该文件命名是以源文件名字命名的。执行此命令时，遇到 import 的包将从GOPATH下面寻找。

  ​		在cmd上这个GOPATH是指环境变量上的GOPATH。

  ​		在goland上，这个GOPATH指项目所设置的GOPATH。

  注意：执行该语句时最好cd到源文件所在目录，否则容易出错。
  
* **go build xxx** ：就是编译xxx，**xxx是一个包含main包的目录，可以是多级目录，只要最终目录包含main包即可**，实际上是GOPATH/src/xxx，与当前所在位置无关。编译后在最终目录下得到编译后的二进制目标执行文件，该文件命名是以最终目录命名的。



# 语法

详见：https://www.runoob.com/go/go-tutorial.html

## 打印

fmt.Print、fmt.Println、fmt.Printf。

* 前面两个就是普通打印

```go
fmt.Println("hello world, I am %d years old\n",28);
```

结果：

hello world, I am %d years old
 28

可见，println不能将字符串中的转义符的值进行带入，把%d当做了普通字符。



* Printf则是按照标准格式打印

```go
fmt.Printf("hello world, I am %d years old\n",28);
```

结果：

hello world, I am 28 years old



fmt还有一个方法 fmt.Sprintf(str, v1,v2,...)。这个方法的使用与 fmt.Printf 一样，不同的是该方法返回格式化字符串，而  fmt.Printf 则是打印格式化字符串。



## 变量声明



### 单变量声明

方式1：

```go
var  variableName  type
```

这是正常的变量声明，即声明一个名为variableName的type类型的变量。



方式2：

```go
var  variableName=value
```

声明一个名为 variableName 的变量，并赋以 value 值，变量类型取决于 value 类型。注意，这是静态类型，所以通过 value 确定类型后，后面就只能存放该类型的值了。



方式3：

```go
variableName := value
```

声明一个名为 variableName 的变量，并赋以 value 值，变量类型取决于 value 类型。注意，这是静态类型，所以通过 value 确定类型后，后面就只能存放该类型的值了。相对于方式2，方式3省略了 var，但是增加了：，：相当于var也就是声明变量的意思。

这种方式比较推荐，因为简短。但只能在函数内部使用，不能用于全局变量的声明。

  

​		注意：全局变量允许不使用，但局部变量声明后必须使用，否则编译错误。



### 多变量声明

```go
//类型相同多个变量, 非全局变量
var vname1, vname2, vname3 type
vname1, vname2, vname3 = v1, v2, v3

var vname1, vname2, vname3 = v1, v2, v3 // 和 python 很像,不需要显示声明类型，自动推断

vname1, vname2, vname3 := v1, v2, v3 // 出现在 := 左侧的变量不应该是已经被声明过的，否则会导致编译错误


// 这种因式分解关键字的写法一般用于声明全局变量
var (
    vname1 type1
    vname2 type2
)
```





## 格式化输出

https://blog.csdn.net/General_zy/article/details/122452868

https://blog.csdn.net/haeasringnar/article/details/84495075



## 交换两个变量的值

这两个变量的类型必须一样，可以通过：a，b=b，a 的方式交换。这是由于go会先计算右边所有值，再复制给左边变量。



## 指针

https://www.cnblogs.com/WORDPAD/p/15016592.html

https://blog.51cto.com/u_6192297/3300583

```go
   var a int= 20   /* 声明实际变量 */
   var ip *int        /* 声明指针变量 */

   ip = &a  /* 指针变量的存储地址 */

   fmt.Printf("a 变量的地址是: %x\n", &a  )

   /* 指针变量的存储地址 */
   fmt.Printf("ip 变量储存的指针地址: %x\n", ip )

   /* 使用指针访问值 */
   fmt.Printf("*ip 变量的值: %d\n", *ip )
```





## 类型大小

https://blog.csdn.net/buptwcx/article/details/107784638



## 数组

go中数组长度必须常量，如果需要某个变量作为长度时，可以把该变量声明为const。



## 结构体

结构体使用与结构体指针的使用方式一样。

但结构体本质还是一种变量，赋值时是值传递，得到的值与原来相等，但不再是同一个内存位置。因此，新旧值在操作时，不会影响对方。

而结构体指针在赋值时虽然也是值传递，但是新旧值都指向同一个结构体，因此操作时会被对方看到。





结构体其实就是go的对象了，因为go里面结构体不但定义了数据，还能定义方法（见接口那一节）。

### 定义数据

```go
type struct_variable_type struct {
   variable1 type1
   variable2 type2
   ...
   variableN typeN
}
```

注意：结构体里面的变量定义不能使用var



### 定义方法

```go
/* 实现接口方法 */
func (struct_name_variable *struct_name) method_name1() [return_type] {
   /* 方法实现 */
}
...
func (struct_name_variable *struct_name) method_namen() [return_type] {
   /* 方法实现*/
}
```

注意：定义结构体方法时，结构体参数必须是结构体指针，而非结构体。

因为单独操作时，结构体和结构体指针并无差别。

但结构体赋值时，得到另外一个结构体，二者操作不会相互影响。

而结构体指针赋值时，得到的指针也是指向原来的结构体。二者操作会被对方感知。



所以为了使得操作能够对调用者产生影响，需要传入结构体指针。





### 创建方式

* 直接创建

```go
variable_name := structure_type {value1, value2...valuen}

variable_name := structure_type { key1: value1, key2: value2..., keyn: valuen}
```

* new 方式创建

```go
variable_name := new(structure_type)
```



### 举例

```go
//定义数据
type Cat struct {
	name string
	age int32
}

//定义方法
func (cat *Cat) sleep() {
	fmt.Println("Cat is sleeping")
}

func (cat *Cat) introduce() {
	fmt.Printf("I am a cat, my name is %v, I am %v years old\n",cat.name,cat.age)
}

func (cat *Cat) add(a int32,b int32) int32 {
	return a+b;
}

//使用
func TestInterface()  {
	cat :=Cat{"lili",13};
	
	cat.introduce()
	cat.sleep();
	sum :=cat.add(35,45)
	fmt.Printf("sum:%v\n",sum)
}
```





## 切片slice

切片类似于Java里的list，指的是变长数组。

另外，切片可以用来创建变量长度数组，如make（[]int,len)，指创建长度为len的切片，并赋初始值为len个0。



### 创建方式

* 通过make方法，如

  ```go
  nums :=make([]int, length, capacity)
  ```

* 直接通过关键字

  ```go
  nums :=[]int{1,2,3,4,5}
  ```



### 获取元素

类似于数组，比如，需要获取index位置的元素，那就是 slice[ index ]。



### 添加元素

append（slice，values...）添加元素。

允许向空的切片（即nil）添加元素。但这个方法返回的是新的切片，不是原来的切片了，也就是说底层是把原来切片复制到一个新的切片，再添加元素。因此，添加元素后需要保存返回的新切片，不然旧切片是没有所添加的元素的。



append方法会影响到slice的以前的append结果，因此，需要保证之前的apend结果都不需要了。



### 复制切片

copy（dst []Type, src []Type）：把src切片复制到dst切片。如果dst切片更长，那么src的值就覆盖dst前面的值，后面的不变。如果dst比src短，那么dst就只能装载src前面的值。



### 长度

len（slice）：切片长度。即切片当下元素个数。



### 容量

cap（slice）：切片容量。



### 多维切片的创建

例如要创建行数为rowN，列数为colN的二维切片

```go
	//先创建行数为rowN的二维切片，这时候每一行是一个长度为0的切片（注意不是nil）
	ans:=make([][] int,rowN) 
	//再逐个创建列数为colN的一维切片，并赋值给相应的行
	for i := 0; i < rowN; i++ {
		ans[i]=make([]int,colN)
	}
	return ans
```





## map

### 创建方式

* 通过make方法

  ```go
  countryCapitalMap := make(map[string]string)
  ```

* 通过关键字

  ```go
  countryCapitalMap := map[string]string{"France": "Paris", "Italy": "Rome"}
  ```



### 插入 k-v

```go
countryCapitalMap [ k ] = v
```


### 删除 k-v

```go
delete(map, k)
```



### 获取 k-v

```go
map[key]  //这种方式获取key对应的value，如果key不存在，则返回value类型的零值
```

如果想知道key是否存在

```go
v,ok=map[key]  //如果key存在，ok为true；否则ok为false
```









## 类型转换

go不支持隐式类型转换，所有类型转换都需要显示声明，如下

```go
 type(variableName)
```



## 接口

### 定义接口

```go
type interface_name interface {
   method_name1 [return_type]
   method_name2 [return_type]
   method_name3 [return_type]
   ...
   method_namen [return_type]
}
```



### 定义结构体

​	定义结构体是为了在结构体里面定义方法，从而使得结构体成为一个对象，如果定义的方法包含接口的所有方法，那就相当于实现了这个接口

```go
/* 定义结构体 */
type struct_name struct {
   /* variables */
}
```



### 实现接口方法

这里是通过在结构体里面定义方法来实现接口方法的，从而实现了接口

```go
/* 实现接口方法 */
func (struct_name_variable struct_name) method_name1() [return_type] {
   /* 方法实现 */
}
...
func (struct_name_variable struct_name) method_namen() [return_type] {
   /* 方法实现*/
}
```



### 使用

创建实现了相应方法的结构体，这时候结构体相当于一个对象。使用方法类似于对象：



## 并发

Go 语言支持并发，我们只需要通过 go 关键字来开启 goroutine 即可。

goroutine 是轻量级线程，goroutine 的调度是由 Golang 运行时进行管理的。

使用方式

```go
go f(x, y, z) //表示开启一个新的线程，执行f函数
```



## 通道

通道可用于两个 goroutine 之间通过传递一个指定类型的值来同步运行和通讯。

通道的读写顺序类似于栈，比如先后写进v1，v2。那么读取的时候，是v2，v1。



### 创建方式

通过make方法来声明一个通道

```go
ch := make(chan int)  //声明一个没有缓冲区的通道
ch := make(chan int, 100) //声明一个缓冲区大小为100的通道
```

如果通道不带缓冲，发送方会阻塞直到接收方从通道中接收了值。如果通道带缓冲，发送方则会阻塞直到发送的值被拷贝到缓冲区内；如果缓冲区已满，则意味着需要等待直到某个接收方获取到一个值。接收方在有值可以接收之前会一直阻塞。



### 发送与接受

操作符 `<-` 用于指定通道的方向，发送或接收。如果未指定方向，则为双向通道。

```go
ch <- v    // 把 v 发送到通道 ch
v := <-ch  // 从 ch 接收数据,并把值赋给 v
```



### 关闭通道

通道的关闭使用close方法

```go
 c := make(chan int, 10)//创建通道
 //各种操作
 close(c)//关闭通道
```



# 常见api

## 打印

```go
	fmt.Println(ans)

	fmt.Print(ans)

	fmt.Printf("...",v1,v2,...) //标准化输出

 	fmt.Sprintf(str, v1,v2,...)  //获取标准化字符串
```



排序

```go
	sort.Ints(nums) //排序整数
```





# 注意

1、当main包有多个文件时，用命令执行main.go可能会报错，解决方法：https://www.cnblogs.com/leafs99/p/golang_main_package.html。



2、当一个module或者包存放在网络上时，我们需要保证module路径（go.mod指明的路径）或者包路径对应其网络存储路径。



3、go除了方法可以跨包调用外，其定义的数据如struct也可以跨包使用，方式与跨包使用方法一样，

即 包名.数据  例如

```go
	cup := _type.Cup{"John", 123};
	fmt.Printf("cup: %v",cup)
```

这里调用了_type包的Cup结构体来创建变量。



4、go语言的方法返回值可以单个也可以多个

```go

func foo1(a string, b int) int {
	fmt.Println("a=", a)
	fmt.Println("b=", b)
 
	c := 100
 
	return c
}
 
//返回多个返回值，匿名的
func fool2(a string, b int) (int, int) {
	fmt.Println("a=", a)
	fmt.Println("b=", b)
 
	return 666, 777
}
 
//返回多个返回值，有形参名称的
func foo3(a string, b int) (r1 int, r2 int) {
	//如果返回值类型都一样的话，还可以这么写
	//func foo3(a string, b int) (r1 ,r2 int) {
	fmt.Println("-------fool3-------")
	fmt.Println("a=", a)
	fmt.Println("b=", b)
	//上面问题的答案
	fmt.Println("r1=", r1) //r1= 0
	fmt.Println("r2=", r2) //r2= 0
 
	r1 = 100
	r2 = 200
	return
}
```



# 待解决问题

1、go的全局变量能够用于不同线程之间的通信吗？

可以的



2、time.Sleep 时间单位

```go
time.Nanosecond //纳秒，=10^(-9)秒
time.Microsecond //微秒，=1000纳秒
time.Millisecond //毫秒，=1000微秒
time.Second //秒，=1000毫秒
```



# 学习进程

1、go path和go vendor  （√）

2、go module  

​     		github的ping，等（为了使用go get）（√）

​			 git的使用（为了看看依赖的依赖怎么来）（√）
