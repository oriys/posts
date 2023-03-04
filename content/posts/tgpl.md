---
title: "笔记::Go编程语言"
date: 2023-02-18T15:36:08+08:00
draft: false
---

## 程序结构

### 名称

`Go` 有 25 个关键字，包括 `if` 和 `switch`，只能在语法允许的情况下使用；不能作为名称使用。

```go
break default func interface select case defer go map struct chan else goto package switch const fallthrough if range type
```

此外，还有大约三十多个预留的名称，如 `int` 和 `true`，用于内置常量、类型和函数。

常量：

```go
true false iota nil
```

类型：

```go
int int8 int16 int32 int64 uint uint8 uint16 uint32 uint64 uintptr float32 float64 complex128 complex64 bool byte rune string error
```

函数:

```go
make len cap new append copy close delete complex real imag panic recover
```

这些名称没有被保留，可以用在声明中，但是要注意潜在可能造成的困惑。  
如果一个实体在一个函数内部声明，只能在函数内部可见。如果它在函数外部声明，可以在包内的任何地方可见。变量名首字母大写表示它是被导出的，可以被包外的代码访问引用，小写表示只能在包内访问。  
变量名没有长度限制，本书建议作用域较小的变量名尽量短，作用域较大的变量名可以考虑长一点使用有意义的名称。  
在风格上，`Go` 使用驼峰命名法。对于 `HTML` 和 `ASCII` 这种专有名词，保持全部大写。

### 声明

有四种主要的声明方式：`var`、`const`、`type` 和 `func`。

```go
package main

import "fmt"

func main() {
	const freezingF, boilingF = 32.0, 212.0
	fmt.Printf("%g°F = %g°C\n", freezingF, fToC(freezingF)) // "32°F = 0°C"
	fmt.Printf("%g°F = %g°C\n", boilingF, fToC(boilingF)) // "212°F = 100°C"
}

func fToC(f float64) float64 {
	return (f - 32) * 5 / 9
}
}
```

一个 `Go` 程序由一个或多个以`.go`为后缀的源文件组成，每个源文件都以一个包声明开始，包声明语句指定了源文件属于哪个包。  
包声明后面是导入声明，然后是包级别的变量、常量、类型和函数声明。  
常量`boilingF`是包级别的常量声明，使用了`const`关键字，关键字可以跟跟一个或多个常量声明。  
函数`fToC`是包级别的函数声明，使用了`func`关键字，关键字后跟函数名、参数列表、返回值列表和函数体。


### 变量

