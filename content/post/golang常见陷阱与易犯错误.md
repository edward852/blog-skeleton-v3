---
title: "Golang常见陷阱与易犯错误"
date: 2021-01-14T13:02:10+08:00
lastmod: 2021-03-14T13:02:10+08:00
draft: false
tags: ["golang"]
categories: ["language"]
mathjax: false
---

本文介绍golang中常见陷阱和易犯错误。  
主要内容是翻译[Traps, Gotchas, and Common Mistakes for New Golang Devs](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang)博文。  
<!--more-->

# 初级
- 左花括号不能单独放一行  
- 存在未使用变量  
  有未使用变量导致编译不通过，注释、删除掉或者赋值给 `_` 即可。  
- 未使用import  
  IDE配置好即可解决，或者用`goimports`工具。  
- 变量简短声明(`:=`)只能在函数内使用  
- 变量简短声明(`:=`)同一block不能单独重复声明已经存在的变量  
  ```go
  one := 0
  one := 1        // error
  one, two := 1,2 // ok
  ```
- 变量简短声明(`:=`)不能作用于结构体字段  
  ```go
  type info struct {
      result int
  }
  
  data.result, err := work() //error
  ```
- 变量Shadowing  
  使用`:=`可能不小心屏蔽了外层同名变量，导致意外的结果。  
  ```go
  x := 1
  {
      x := 2
      fmt.Println(x) //prints 2
  }
  fmt.Println(x)     //prints 1
  ```
- 不能使用nil初始化未指定类型的变量  
  ```go
  var x = nil //error
  ```
- 直接使用nil的slice和map  
  一般make之后再使用。  
  ```go
  var s []int
  s = append(s,1) // ok
  
  var m map[string]int
  m["one"] = 1    // error
  ```
- map容量  
  make map时可以指定容量，不过不能对其调用cap。  
- 字符串不能赋值为nil  
  ```go
  var s1 string = nil // error
  var s2 string       // ok, defaults to "" 
  ```
- 数组类型函数参数  
  (静态)数组作为函数参数是值拷贝，函数内的修改不作用于原来的数组。要修改可以用slice或指针。  
- range遍历slice或map  
  range遍历slice或map时遍历变量第一个是索引，第二个才是数据(值拷贝)。  
- slice和数组是一维的  
  动态多维数组可以模拟，但是使用上不是很方便。  
  ```go
  x := 2
  y := 4

  table := make([][]int,x)
  for i:= range table {
      table[i] = make([]int,y)
  }
  ```
- 读取map中不存在的键值  
  不返回nil而是默认零值。判断键是否存在需要检查第二个返回值。  
- 字符串是只读的  
  不能更新字符串中的字符，需要转换成字节slice才能操作。  
- 字符串和字节slice间的转换  
  转换是完整的拷贝，而不是其它语言中的强制类型转换。  
- 字符串索引访问  
  返回的是字节而不是字符(可能是多个字节)。  
  需要访问字符的话可以用`for range`语句或者`utf8`包。  
- 字符串并不总是UTF-8编码  
  字符串字面量是UTF-8编码，字符串变量可以是任意编码(如UTF-16)。  
- 字符串长度  
  len函数返回的是字节数而不是字符数。字符数可以通过对应编码包的接口获取。  
- 多行slice、数组、map字面量遗漏逗号  
  ```go
  x := []int{
      1,
      2 //error
  }
  ```
- log.Fatal和log.Panic不仅输出日志，还会终止程序  
- 大部分内置数据类型操作并不是并发安全的  
  需要channel或者sync包确保并发安全。  
- range遍历字符串  
  索引是每个字符第一个字节的索引。默认按照UTF-8编码解析。  
- range遍历map  
  遍历顺序并不固定。  
- switch语句fall through  
  case分支默认break而不是fall through。  
  需要fall through可以使用fallthrough或写在同一case。  
- 自增、自减  
  是语句而不是表达式，没有值，不能用于表达式中。  
- 取反、异或运算符都是`^`  
- 运算符优先级差异  
  不确定就加括号。  
- 未导出结构体字段不会编码  
  小写字段不会编码。  
- 程序退出时仍有活跃协程  
  可以通过channel、WaitGroup(不能复制！)等手段等待其它协程都结束才退出。  
- 发送数据到已关闭channel会panic  
- 使用nil的channel  
  对nil的channel进行收、发操作会导致阻塞、死锁。  
  配合select使用可以实现开启、关闭特定case分支的效果。  
  ```go
  inch := make(chan int)
  outch := make(chan int)

  go func() {
      var in <- chan int = inch
      var out chan <- int
      var val int
      
      for {
          select {
          case out <- val:
              out = nil
              in = inch
          case val = <- in:
              out = outch
              in = nil
          }
      }
  }()
  ```
- 传值接收者方法不会修改接收者的值  
  需要传指针才可以修改。引用类型的数据结构(如slice, map)本质上就是传指针。  

# 中级
- 关闭HTTP Response Body  
  ```go
  resp, err := http.Get("https://golang.org")
  if resp != nil {
      defer resp.Body.Close()
  }
  if err != nil {
      // error handling
      return
  }
  ```
- 关闭HTTP连接  
  ```go
  req, err := http.NewRequest("GET", "https://golang.org", nil)
  if err != nil {
      // 处理出错情况
      return
  }
  
  req.Close = true
  // 或者这样处理：
  //req.Header.Add("Connection", "close")
  ```
- 官方JSON编码器会自动添加`\n`符  
- 官方JSON包默认会转义特殊HTML字符  
- 官方JSON反序列化(unmarshal)数字默认类型为`float64`，包括整数
- 官方JSON字符串值不能有十六进制或者非UTF-8编码转义序列  
- 比较结构体、数组、slice和map  
  相同类型且元素或所有字段本身可比较的数组或结构体才能用`==`比较，slice、map不能比较。  
  不能`==`比较则使用helper函数(如`reflect.DeepEqual`,`bytes.Equals`,`strings.EqualFold`)比较。  
- 从panic恢复  
  在defer的函数里**直接调用**recover才能从panic恢复。  
  ```go
  func doRecover() {  
      fmt.Println("recovered =>", recover()) //prints: recovered => <nil>
  }
  
  func submit() {
      defer func() {
          doRecover() //panic is not recovered
      }()
      
      // ...
      panic(err)
  }
  ```
- range语句中更新、引用slice，数组，map的元素值  
  循环变量只是元素的值拷贝，对其取地址、更新都不会符合编码预期。  
  通过索引运算符才能正确取地址、更新元素值。元素直接存指针也可以(增加GC负担)。  
  ```go
  data := []int{1,2,3}
  for _, v := range data {
      v *= 10 // original item is not changed
      //data[i] *= 10
  }
  
  fmt.Println("data:", data) //prints data: [1 2 3]
  ```
- slice中隐藏的数据  
  对slice切片返回的新slice会共享其底层数组，可能导致错误的内存使用(读取原slice其它内容)。  
  ```go
  raw := make([]byte, 64)
  copy(raw, "public secret")
  field := raw[:6]

  fmt.Println("field:", string(field))
  fmt.Println("all:", string(field[:cap(field)]))
  ```
  通过copy需要的数据来避免上述情况。  
  ```go
  func getField() []byte {
      raw := make([]byte, 10000)
      res := make([]byte, 6)
      
      copy(res, raw[:6])
      
      return res
  }
  ```
- slice数据污染  
  对slice切片返回的新slice会共享其底层数组，可能导致错误的内存使用(修改原slice内容)。  
  ```go
  path := []byte("AAAA/BBBBBBBBB")
  sepIndex := bytes.IndexByte(path, '/')
  dir1 := path[:sepIndex]
  dir2 := path[sepIndex+1:]
	
  fmt.Println("dir1 =>", string(dir1)) // prints: dir1 => AAAA
  fmt.Println("dir2 =>", string(dir2)) // prints: dir2 => BBBBBBBBB

  dir1 = append(dir1, "suffix"...)
  fmt.Println("dir2 =>", string(dir2)) // prints: dir2 => uffixBBBB
  ```
  可以采取拷贝副本的方法解决，也可以通过切片操作的第3个参数控制容量。  
  `dir1 := path[:sepIndex:sepIndex]`写超过容量会触发分配新的缓冲区而不是部分覆盖`dir2`。  
- 过期的slice  
  多个slice共享同一底层数组，发生扩容时会导致其它slice指向旧的数据。  
- 类型声明和方法  
  基于已有类型(非接口类型)声明新类型，新类型不能调用原有类型的方法。  
  ```go
  type myMutex sync.Mutex
  
  var mtx myMutex
  mtx.Lock() // error
  ```
  可以通过匿名字段嵌入的方式得到(不是继承，是组合！)原类型的方法：  
  ```go
  type myLocker struct {  
      sync.Mutex
  }
  
  var lock myLocker
  lock.Lock() //ok
  ```
- 跳出`for switch`和`for select`代码块  
  通过label跳出外部循环。  
  ```go
  loop:
      for {
          switch {
          case true:
              fmt.Println("breaking out...")
              break loop
          }
      }
  
      fmt.Println("out!")
  ```
- for循环中的迭代变量和闭包  
  每轮循环都是同一迭代变量，赋予新的值。  
  闭包捕获同一变量，很可能不符合编程预期。  
  ```go
  data := []string{"one", "two", "three"}

  for _, v := range data {
      go func() {
          fmt.Println(v)
      }()
  }
  
  time.Sleep(3 * time.Second)
  // goroutines print: three, three, three
  ```
  解决方法一：赋值给循环体内新变量：  
  ```go
  for _, v := range data {
      vcopy := v
      
      go func() {
        fmt.Println(vcopy)
      }()
  }
  ```
  解决方法二：迭代变量作为参数传递给闭包：  
  ```go
  for _, v := range data {
      go func(in string) {
        fmt.Println(in)
      }(v)
  }
  ```
  比较隐蔽的情况(接收者本质就是参数)：  
  ```go
  type field struct {
      name string
  }
  
  func (p *field) print() {
      fmt.Println(p.name)
  }
  
  data1 := []field{{"one"}, {"two"}, {"three"}}  // values
  data2 := []*field{{"one"}, {"two"}, {"three"}} // pointers
  
  for _, v := range data1 {
      go v.print()
  }
  time.Sleep(1 * time.Second)
  
  for _, v := range data2 {
      go v.print()
  }
  time.Sleep(1 * time.Second)
  ```
- defer函数参数求值时机  
  defer语句求值时，defer函数的参数就会求值(而不是函数运行时)。  
- defer函数执行时机和顺序  
  defer函数在所在函数即将结束返回时执行，有多个defer函数则以后进先出顺序执行。  
  在for循环体中defer释放资源操作不是合适的做法，应该包装到函数内或者用完直接释放资源。  
  ```go
  for _, target := range targets {
        func() {
            f, err := os.Open(target)
            if err != nil {
                fmt.Println("bad target:",target,"error:",err)
                return
            }
            
            defer f.Close() //ok
            //do something with the file...
        }()
  }
  ```
- 类型断言失败  
  类型断言失败返回对应断言类型的零值。再加上变量shadowing就会产生更迷惑的效果。  
  ```go
  var data interface{} = "confusing"
  
  if data, ok := data.(int); ok {
      fmt.Println("[is an int] value =>",data)
  } else {
      fmt.Println("[not an int] value =>",data) 
      //prints: [not an int] value => 0 (not "confusing")
  }
  ```
- 阻塞的协程与资源泄露  
  从数据库集群拉取最先返回的结果，可行但是有bug的实现如下：  
  ```go
  func First(query string, replicas ...Search) Result {
      c := make(chan Result)
      searchReplica := func(i int) { c <- replicas[i](query) }
      
      for i := range replicas {
          go searchReplica(i)
      }
      
      return <-c
  }
  ```
  由于channel无缓冲，所以最快的协程正常退出，其他都永久阻塞在返回结果那一步，造成资源泄露。  
  解决方法一：创建有足够缓冲的channel：`c := make(chan Result, len(replicas))`  
  解决方法二：`selece`+`default`+1格缓冲的channel：  
  ```go
  func First(query string, replicas ...Search) Result {
      c := make(chan Result, 1)
      searchReplica := func(i int) {
          select {
          case c <- replicas[i](query):
          default:
          }
      }
      
      for i := range replicas {
          go searchReplica(i)
      }
      
      return <-c
  }
  ```
  解决方法三：关闭channel通知协程：  
  ```go
  func First(query string, replicas ...Search) Result {  
      c := make(chan Result)
      done := make(chan struct{})
      defer close(done)
      searchReplica := func(i int) { 
          select {
          case c <- replicas[i](query):
          case <- done:
          }
      }
      
      for i := range replicas {
          go searchReplica(i)
      }
      
      return <-c
  }
  ```
- 不同的0字节变量分配相同的地址  
- 首次使用iota并不总是从0开始  
  iota是const定义块内当前行的索引，不是第一行就不是0。  
  ```go
  const (
      azero = iota
      aone  = iota
  )
  const (
      info  = "processing"
      bzero = iota
      bone  = iota
  )
  
  fmt.Println(azero, aone) //prints: 0 1
  fmt.Println(bzero, bone) //prints: 1 2
  ```

# 高级
- 值实例调用指针接收者方法  
  只要值本身是可取地址，就可以调用指针接收者方法。  
  map的元素不可取地址，接口引用的变量也是不可取地址的。  
  ```go
  type data struct {
      name string
  }
  func (p *data) print() {
      fmt.Println("name:", p.name)
  }
  type printer interface {
      print()
  }
  
  d1 := data{"one"}
  d1.print() // 合法
  
  var in printer = data{"two"} // 错误。 *data实现了printer接口，但是data没有
  in.print()
  
  m := map[string]data {"x":data{"three"}}
  m["x"].print() // 错误，map的元素不可取地址
  ```

- 更新map的值类型元素  
  如果map的元素是结构体(非指针类型)，那么不能单独更新结构体的字段值。  
  这是因为map的元素不可取址。  
  不过令人迷惑的是slice支持这样的操作。  
  ```go
  type data struct {  
      name string
  }
  m := map[string]data {"x":{"one"}}
  m["x"].name = "two" // error
  
  s := []data {{"one"}}
  s[0].name = "two" // ok
  fmt.Println(s)    // prints: [{two}]
  ```
- nil接口和nil接口值  
  接口变量为nil当且仅当其type和value字段都为nil。  
  ```go
  var data *byte
  var in interface{}
  
  fmt.Println(data, data == nil) // prints: <nil> true
  fmt.Println(in, in == nil)     // prints: <nil> true
  
  in = data
  fmt.Println(in, in == nil)     // prints: <nil> false
  // 'data' is 'nil', but 'in' is not 'nil'
  ```
  返回接口类型的函数尤其需要注意这种情况，明确地`return nil`。  
- 栈和堆变量  
  跟C++不同，变量分配在堆上和栈上不由关键字决定。  
  Go会根据变量内存占用大小以及逃逸分析(可以参考[这里](/post/golang逃逸分析与优化))决定将变量分配到哪。  

- GOMAXPROCS, Concurrency和Parallelism  
  具体参考[runtime](https://golang.org/pkg/runtime)的说明以及Go的调度模型。  
  > The GOMAXPROCS variable limits the number of operating system threads that can execute user-level Go code simultaneously. There is no limit to the number of threads that can be blocked in system calls on behalf of Go code; those do not count against the GOMAXPROCS limit. 

- 读写操作(指令)重排  
  Go编译器会在保证协程内(不包括协程间)最终结果的前提下重排读写指令(现代CPU也会指令重排)。  
  比如说u1内b并不依赖a，所以可能先执行`b = 2`然后再执行`a = 1`。  
  ```go
  var a, b int
  
  func u1() {
      a = 1
      b = 2
  }
  func u2() {
      a = 3
      b = 4
  }
  func p() {
      println(a)
      println(b)
  }
  
  func main() {
      go u1()
      go u2()
      go p()
      
      time.Sleep(1 * time.Second)
  }
  ```
- 抢占式调度  
  在老版本Go(单线程)中如果有协程死循环而且没有触发(显式或隐式)调度的代码，那么程序就会死锁。  
  ```go
  var _ = runtime.GOMAXPROCS(1) // 设置成1新版本Go下面的代码才会死锁
  
  func main() {
      done := false
      
      go func(){
        done = true
      }()
      
      for !done {
          // 空或者没有触发调度的代码
        
          //time.Sleep(time.Second)
		  //runtime.Gosched()
      }
      
      fmt.Println("done!")
  }
  ```
  解决方法是for循环里面显式调用`runtime.Gosched()`或者其他阻塞操作(如`time.Sleep`)。  
  当然新版本的Go和现代CPU默认情况下并不需要特别处理也不会死锁。  

- import "C"和import块  
  需要imort C包才能使用cgo，不能和其他包混在一起写。  
  ```go
  /*
  #include <stdlib.h>
  */
  import (
      "C"
  )
  
  import (
      "unsafe"
  )
  ```
- import "C"和cgo注释之间不能有空行  
- 不能调用C语言变参函数  
  需要改为定参函数。  

