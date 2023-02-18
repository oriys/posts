---
title: "笔记::Go编程语言"
date: 2023-02-18T15:36:08+08:00
draft: false
---

## 程序结构

### 名称

Go 有 25 个关键字，包括 if 和 switch，只能在语法允许的情况下使用；不能作为名称使用。

```go
break default func interface select case defer go map struct chan else goto package switch const fallthrough if range type
```

此外，还有大约三十多个预留的名称，如 int 和 true，用于内置常量、类型和函数。

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

