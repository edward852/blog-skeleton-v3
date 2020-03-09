---
title: "golang反射的使用"
date: 2019-08-10T19:37:35+08:00
lastmod: 2019-08-10T19:37:35+08:00
draft: false
tags: ["golang", "reflect"]
categories: ["language"]
mathjax: false
---

本文主要参考 `The Laws of Reflection` [^1] 介绍反射的使用。  
<!--more-->

# 类型和接口
golang的类型是静态的，在编译的时候每个变量的类型就确定了，下文以 `静态类型` 指代。  
静态类型包括用户自定义的类型，细分下来，是由基本类型组成的。  

接口是静态类型中一种比较重要的类别，是固定数目方法的集合。  
空接口 `interface{}` 更是特别的接口，不要求实现任何方法，任意值都满足，因此可以表示任意值。  

# 接口表示
接口的底层实现可以参考 [这篇文章](https://research.swtch.com/interfaces) ，不过可以简单理解为一个 `pair` ，包括 `具体值` 和 `具体类型信息`。  
具体值就是变量的值，具体类型信息可以理解成变量的静态类型(不是对应的接口类型)。  
```go
type Stringer interface {
	String() string
}

type Intf struct {
	Name string
}

func (i Intf) String() string {
	return i.Name
}
```
以上面代码为例，Intf类型变量可以赋值给空接口或者Stringer接口类型变量。  
不过实际上存储都是具体的Intf变量值以及Intf类型信息。  
之所以要存具体类型信息，是因为只知道接口类型的话没法确定对应实现的方法。  
接口类型+具体类型才能确定实现接口的具体方法列表(运行时算出并缓存)。  

# 从接口获取反射对象
从接口的表示我们知道，接口包括值和类型，所以反射之后也分为反射类型和反射值。  

通过 `reflect.TypeOf` 我们能获取接口的反射类型。  
通过 `reflect.ValueOf` 我们能获取接口的反射值。  
通过 `Kind` 方法我们能知道存储数据的底层类型/基本类型(Chan, Func, Ptr, Slice...)。  
反射值有许多方法可以获取实际的值，如 `Float` 。  

# 从反射对象获取接口
通过 `Interface` 方法，我们能从反射值获取接口，为空接口类型。  
```go
func (v Value) Interface() interface{}
```
空接口可以通过类型转换或者switch按类型处理。  
```go
func ToString(any interface{}) string {
    // 类型转换
    if v, ok := any.(Stringer); ok {
        return v.String()
    }
    
    // switch按类型处理
    switch v := any.(type) {
    case int:
        return strconv.Itoa(v)
    case bool:
        return strconv.FormatBool(v)
    }
    
    return "???"
}
```

# 反射如何修改值
反射值分为只读类型和可设置类型，通过 `CanSet` 方法可以判断。  
类比C语言，我们知道传值与传指针的区别，那就是传指针才能修改实参。  
类似的，指针对应的反射值才是可设置的。  
变量指针的反射值通过 `Elem` 方法获取指针指向的、可设置的反射值。  
```go
var f float64 = 3.4

p := reflect.ValueOf(&f) // f的指针
fmt.Println("type of p:", p.Type())
// 输出 *float64
fmt.Println("settability of p:", p.CanSet())
// 输出 false

v := p.Elem()
fmt.Println("settability of v:", v.CanSet())
// 输出 true
v.SetFloat(7.1)

fmt.Println(f)
// 输出 7.1
```

## 修改结构体字段值
通过结构体指针的反射值，我们能够获取其指向的实际结构体变量的反射值，进而修改任意字段值。  
```go
type T struct {
	A int
	B string
}

t := T{23, "skidoo"}
s := reflect.ValueOf(&t).Elem()
typeOfT := s.Type()
for i := 0; i < s.NumField(); i++ {
	f := s.Field(i)
	fmt.Printf("%d: %s %s = %v\n", i, typeOfT.Field(i).Name, f.Type(), f.Interface())
}

s.Field(0).SetInt(77)
s.Field(1).SetString("Sunset Strip")
fmt.Println("t is now", t)
```

[^1]: https://blog.golang.org/laws-of-reflection
