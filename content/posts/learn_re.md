---
title: "学习正则表达式"
date: 2023-02-28T00:03:05+08:00
draft: false
---

## 字符组

### 普通字符组

字符组是一组字符，在正则表达式中，它表示“在同一个位置可能出现的各种字符”，其写法是在一对方括号[和]之间列出所有可能出现的字符，简单的字符组包括[ab]、 [314]、[#.?]等。

普通字符串匹配数字字符的正则表达式：

`[0123456789]`

### 范围字符组

正则表达式提供了`-范围表示法(range)`，它更直观，能进一步简化字符组。所谓`-范围表示法`，就是用 麻烦，这样 就可以表示为确实比的形式表示 x 到 y 整个范围内的字符，省去一一列出的麻烦。
在字符组中，-表示的范围，一般是根据字符对应的码值(Code Point，也就是字符在对应编码表中的编码的数值)来确定的，码值小的字符在前，码值大的字符在后。因此[0-9]
是合法的，[9-0]会报错。

范围字符串匹配数字字符的正则表达式：

`[0-9]`

在字符组中可以同时并列多个`-范围表示法`，字符组[0-9a-zA-Z]可以匹配数字、大写 字母或小写字母

表示十六进制

`[0-9a-fA-F]`

### 字符串转义

在字符组中，如果要匹配的字符是特殊字符，就需要对特殊字符进行转义，比如要匹配字符组中的`-`，就需要对`-`进行转义，即`\-`。但是有时候却不需要转义。

如果要在字符组中匹配`-`，最好的办法是把`-`放在字符组的最前面或者最后面，这样就不需要转义了。比如，要匹配`-`，可以写成`[0-9-]`或者`[-0-9]`。

同理，如果要字符组中匹配方括号，则需要在方括号前面加上反斜杠`\`进行转义，即`[123\]567]`如果要匹配`[123]567`，则正则表达式为`\[123\]567]`，末尾的`]`不需要转义。

### 取反字符组

在字符组中，如果在第一个字符前面加上`^`，则表示取反，即不匹配字符组中的字符。比如，`[^0-9]`表示不匹配数字字符，`[^a-z]`表示不匹配小写字母。取反字符组必须匹配一个字符，如果不匹配任何字符，则会报错。

匹配0,9之外的数字

java

`[^09]`

`^`是正则表达式的元字符，只在`[`后面才表示取反，如果`^`不在`[`后面，匹配时不需要转义。 `[0^12]`和`[\^012]`是等价的，都表示匹配字符`0`、`^`或`1`、`2`。但是后者写起来会麻烦一些。

### 字符组简记法

对于常用的字符组，正则表达式提供了简记法，比如`\d`表示数字字符，`\w`表示单词字符，`\s`表示空白字符。

| 字符组 | 简记法 |完整表示|
|------|-----|--|
| 数字字符 | \d | [0-9] |
| 单词字符 | \w | [a-zA-Z0-9_] |
| 空白字符 | \s | [ \t\n\v\f\r] |

`\w` 匹配单词字符，即字母、数字或下划线。如果只要匹配字母和数字，可以使用`[a-zA-Z0-9]`。

简记法也可以和普通字符组、范围字符组、取反字符组混合使用。

用在普通字符串内部

`[\da-zA-Z]`表示匹配数字字符或字母字符。

用在取反字符组内部

`[^\w]`表示匹配非单词字符。

`\D`, `\W`, `\S` 的使用

`\D`、`\W`、`\S`分别表示非数字字符、非单词字符、非空白字符。他们与`\d`、`\w`、`\s`形成互补关系。

匹配所有字符

`[\s\S]`,`\s`表示空白字符，`\S`表示非空白字符，因此`[\s\S]`表示所有字符。同理，`[\d\D]` 和 `[\w\W]`表示所有字符。

如果在字符组中使用了简记法，则最好不要出现范围字符组，比如`[\d-a]`，简记法和范围字符组混用会导致不可预知的结果。

### 字符组运算

有部分编程语言提供了字符组运算的支持。不同的编程语言支持的字符组运算符不同，比如Java中的字符组运算符是`&&`，而Go中的字符组运算符是`-`。

我们要实现在字母中排除元音字母的效果，按照之前的写法，需要写成`[bcdfghjklmnpqrstvwxyz]`或者`[b-df-hj-np-tv-z]`，如果使用字符组运算，可以写成`[a-z&&[^aeiou]]`
。在这里，`&&`表示字符组运算符，`&&`前面的字符组表示基础字符组，`&&`后面的字符组表示排除字符组，即基础字符组中排除排除字符组中的字符。两者取交集，得到最终的字符组。

### POSIX 字符组

以上的字符组定义都属于`PCRE`的流派，`PCRE`是Perl语言的正则表达式引擎，它的正则表达式语法和Perl语言的正则表达式语法基本一致，在大多数语言中得到了支持。
POSIX字符组是`POSIX`的流派，`POSIX`
是一个操作系统标准，它定义了操作系统的接口和行为，比如Linux/Unix操作系统都是基于`POSIX`标准的。
它的语法和普通字符组不同，但是功能和普通字符组一样。POSIX字符组的语法是`[:name:]`，其中`name`
是字符组的名称。他主要的的使用场景是Linux/Unix环境下各种工具（比如grep、sed、awk等）的正则表达式。

| POSIX字符组 | 说明 | PCRE|
|------|-----|--|
| [:alnum:] | 字母或数字 | [a-zA-Z0-9] |
| [:alpha:] | 字母 | [a-zA-Z] |
| [:blank:] | 空白字符 | [ \t] |
| [:cntrl:] | 控制字符 | [\x00-\x1F\x7F] |  
| [:digit:] | 数字 | [0-9] |
| [:graph:] | 图形字符 | [!-~] |
| [:lower:] | 小写字母 | [a-z] |
| [:print:] | 可打印字符 | [ -~] |
| [:punct:] | 标点符号 | [!-/:-@[-`{-~] |
| [:space:] | 空白字符 | [ \t\n\v\f\r] |
| [:upper:] | 大写字母 | [A-Z] |
| [:word:] | 单词字符 | [a-zA-Z0-9_] |
| [:xdigit:] | 十六进制数字 | [0-9a-fA-F] |

不同语言对POSIX字符组的支持情况不同，视具体情况而定。